# Docker Ruby Notes

## Running a Rails server in a Docker container

Add this to your app's directory: `Dockerfile`

```Dockerfile
FROM ruby:2.6
RUN apt-get update -yqq \
    && apt-get install -yqq --no-install-recommends \
    nodejs
COPY . /usr/src/app
WORKDIR /usr/src/app
RUN bundle install
CMD ["bin/rails", "s", "-b", "0.0.0.0"]
```

The `-b 0.0.0.0` tells the rails server to accept traffic from any location, not just localhost.  Since we're running in a container, the traffic coming from our host machine to the Docker container will not be localhost.

Build the image from this Dockerfile:

```bash
$ docker build . -t myapp:1.0 -t myapp
```

Here I'm creating multiple tags, a versioned tag and an easy to use generic tag.

After the command finishes, you should see the unique ID assigned to it.

Run a container from the image:

```bash
$ docker run -p 1234:3000 myapp
```

This will run the Rails app in the container and make it availabe on your host machine at port 1234.

This results in an image that is just over 1G.  The base Ruby image uses Ubuntu.  For a smaller image you can use the Alpine Linux based Ruby image:

```Dockerfile
FROM ruby:2.6-alpine
RUN apk add --update \
    build-base \
    postgresql-dev \
    sqlite-dev \
    tzdata \
    bash \
    nodejs \
    yarn \
    git \
    && rm -rf /var/cache/apk/*

ENV BUNDLE_PATH /gems

COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app
RUN bundle install

COPY package.json /usr/src/app/
COPY yarn.lock /usr/src/app/
RUN yarn install

COPY . /usr/src/app/
ENTRYPOINT [ "./docker-entrypoint.sh" ]
```

This produces an image that is about 350MB.

## Ignoring files from the image build

You don't want to include some of the file from your app when you build the image.  To hide files we use a `.dockerignore` file.

```txt
.git
.gitignore
log/*
tmp/*
```

## Avoiding bundle install every time you edit a file

Let's say you have this config:

```Dockerfile
FROM ruby:2.6-alpine
RUN apk add --update \
    build-base \
    postgresql-dev \
    sqlite-dev \
    nodejs \
    tzdata \
    bash \
    && rm -rf /var/cache/apk/*
COPY . /usr/src/app/
WORKDIR /usr/src/app/
RUN bundle install
CMD ["bin/rails", "s", "-b", "0.0.0.0"]
```

If you change the README in your app and then rebuild, the gems get installed again, which is slow.  Docker will cache the layers used to create each step of the build process, but we've updated a file causing the caches for steps after the `COPY` command to be busted.  So, bundle install needs to get run again.

To get around this, copy and the `Gemfile` and `Gemfile.lock` over first, run a bundle install and then copy the rest of the app files over.  This way the cache for the bundle install only needs to get run.

## Docker composing

Given a `docker-compose.yml` file:

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - '3000:3000'
    volumes:
      - .:/usr/src/app
    env_file:
      - .env/development/web
      - .env/development/database
  redis:
    image: redis
  database:
    image: postgres
    env_file:
      - .env/development/database
    volumes:
      - db_data:/var/lib/postgresql/data
volumes:
  db_data:
```

### Run it

You can start up the cluster of containers with:

```bash
$ docker-compose up
```

### Verify that the containers are running

You can view the running containers with:

```bash
$ docker-compose ps
```

### Access Redis

You can log into the Redis container with:

```bash
$ docker-compose run --rm redis redis-cli -h redis
```

This creates a new throw-away container based on the redis image and runs the `redis-cli` command with the `-h` or host flag pointing at the already running redis container.  The magic of docker compose's networking means the running redis container can just be refered to by the name "redis".

### Access Postgres

```bash
$ docker-compose run --rm postgres psql -U postgres -h postgres
```

After entering the password you should be in.

### Rebuild to add in new gems

After you add a new gem to the Gemfile, to run `bundle install` you need to rebuild the image:

```bash
# stop the container
$ docker-compose stop web
# add gems to the Gemfile
# build the image, which will do the bundle install
$ docker-compose build web
# restart the web image
$ docker-compose up -d --force-recreate web
```

`--force-recreate` = Recreate containers even if their configuration and image haven't changed.

### Generate a model and migrate

```bash
$ docker-compose exec web bin/rails g model User first_name:string last_name:string
$ docker-compose exec web bin/rails migrate
```

### Find out information about the database volume

```bash
$ docker volume inspect myapp_db_data
# [
#     {
#         "CreatedAt": "2019-01-21T19:36:28Z",
#         "Driver": "local",
#         "Labels": {
#             "com.docker.compose.project": "myapp",
#             "com.docker.compose.version": "1.23.2",
#             "com.docker.compose.volume": "db_data"
#         },
#         "Mountpoint": "/var/lib/docker/volumes/myapp_db_data/_data",
#         "Name": "myapp_db_data",
#         "Options": null,
#         "Scope": "local"
#     }
# ]
```

If you are not sure of the name of the volume:

```bash
$ docker volume ls
```

### Add webpacker and rails to an existing app

It should be this simple, but the following was not sufficient when I tried it.  There was hours of troubleshooting and magic webpack/babel encantations.  Hopefully in the future, the following will suffice:

If the web container is running, stop it:

```bash
$ docker-compose stop web
```

Add the webpacker gem to your `Gemfile`:

```ruby
gem 'webpacker'
```

Build the image to run the bundle install command:

```bash
$ docker-compose build web
```

Install webpacker:

```bash
$ docker-compose run web bin/rails webpacker:install
```

Install React:

```bash
$ docker-compose run web bin/rails webpacker:install:react
```

### Add a binding pry to it

You need to make sure that your web service has the following config in the docker compose:

```yaml
stdin_open: true
tty: true
```

Then you can do your standard `$ docker-compose up` command.

In another terminal, attach with:

```bash
$ docker attach $(docker-compose ps -q web)
```

When a `binding.pry` is hit, the prompt will show up in this window.

In order for this to work, I needed to add the `tput` command, which is a part of the `ncurses` package:

```bash
$ apk add ncurses
```

## Integrating CI

At Semaphore CI.  The steps are:

1. Push to master on GitHub gets picked up by Semaphore
1. Semaphore grabs the code and runs the specs
1. When the specs pass, it builds a production Docker image for the app
1. Semaphore pushes the new image to the Docker Hub
1. It then initiates a Swarm stack deploy on the Swarm master node

I'm using the following build commands:

```bash
$ mv /home/runner/onehouse-website-rails/config/database.production.yml /home/runner/onehouse-website-rails/config/database.yml
$ docker build -f Dockerfile.production -t elliotlarson/onehouse-website:production .
$ docker push elliotlarson/onehouse-website:production
$ export DOCKER_TLS_VERIFY="1"
$ export DOCKER_HOST="tcp://178.128.15.86:2376"
$ export DOCKER_CERT_PATH="/home/runner/onehouse-website-rails/.docker/"
$ docker stack deploy --prune --with-registry-auth -c docker-stack.yml ohweb
$ docker system prune --force
```

When Semaphore grabs the code and runs the spec suite it replaces the `database.yml` file with its own.  Before we build the Docker image, we need to replace this file with our own.  The `database.production.yml` file lives in the repo and is copied over.  The file uses environment variables for passwords, etc.

The `docker-stack.yml` file looks like:

```yaml
version: '3'
services:

  web:
    image: elliotlarson/onehouse-website:production
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
    ports:
      - 80:3000
    env_file:
      - .env.production

  database:
    image: postgres:11.1-alpine
    env_file:
      - .env.production
    volumes:
      - db_data:/var/lib/postgresql/data

  db-creator:
    image: elliotlarson/myapp_web:prod
    command: ["bin/rails", "db:create"]
    env_file:
      - .env.production
    deploy:
      restart_policy:
        condition: none
    depends_on:
      - database

  db-migrator:
    image: elliotlarson/myapp_web:prod
    command: ["bin/rails", "db:migrate"]
    env_file:
      - .env.production
    deploy:
      restart_policy:
        condition: none
    depends_on:
      - db-creator

volumes:
  db_data:
  gem_cache:
```

It points to a `.env.production` file.  This is added to Semaphore as an encrypted configuration file.  When Semaphore grabs the code, it also puts the configuration files in place.

In Semaphore we also setup a Docker Hub integration.  This allows us to push to the Docker Hub after building the image without having to login each time.

In order to deploy to the Docker Swarm on the server I needed to add the appropriate certificates.  These you can get from the `docker-machine`:

```bash
$ docker-machine config ohweb
# --tlsverify
# --tlscacert="/Users/elliot/.docker/machine/machines/ohweb/ca.pem"
# --tlscert="/Users/elliot/.docker/machine/machines/ohweb/cert.pem"
# --tlskey="/Users/elliot/.docker/machine/machines/ohweb/key.pem"
# -H=tcp://178.128.15.86:2376
```

This shows you where the certs are.  I copied these and created encrypted configuration files with them in Semaphore:

* `.docker/ca.pem`
* `.docker/cert.pem`
* `.docker/key.pem`

The environment variable in the build commands `DOCKER_CERT_PATH` is set to look in this directory.

### Checking deploy status

If you want to see when the last time a service was updated you can:

```bash
$ eval $(docker-machine env ohweb)
$ docker service ps ohweb_web
# fueeao0m5boy        ohweb_web.1         elliotlarson/onehouse-website:production   ohweb               Running             Running 44 seconds ago
# rp0m280pwdw7         \_ ohweb_web.1     elliotlarson/onehouse-website:production   ohweb               Shutdown            Shutdown 45 seconds ago
# e28w7sho9xb7         \_ ohweb_web.1     elliotlarson/onehouse-website:production   ohweb               Shutdown            Shutdown 14 hours ago
# wwbrrcm40bx4         \_ ohweb_web.1     elliotlarson/onehouse-website:production   ohweb               Shutdown            Shutdown 14 hours ago
# bk4oaalfpg57         \_ ohweb_web.1     elliotlarson/onehouse-website:production   ohweb               Shutdown            Shutdown 14 hours ago
# qrsk9g7v86pt        ohweb_web.2         elliotlarson/onehouse-website:production   ohweb               Running             Running 59 seconds ago
# beolmpmpeeoj         \_ ohweb_web.2     elliotlarson/onehouse-website:production   ohweb               Shutdown            Shutdown about a minute ago
# w5i7edpwlrkd         \_ ohweb_web.2     elliotlarson/onehouse-website:production   ohweb               Shutdown            Shutdown 14 hours ago
# pvkxqez0sbud         \_ ohweb_web.2     elliotlarson/onehouse-website:production   ohweb               Shutdown            Shutdown 14 hours ago
# w07ccgyulfhe         \_ ohweb_web.2     elliotlarson/onehouse-website:production   ohweb               Shutdown            Shutdown 14 hours ago
```

Notice that the first entry of the output shows the service was updated 44 seconds ago.
