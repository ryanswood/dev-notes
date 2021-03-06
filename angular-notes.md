# Angular Notes

I'm starting to learn Angular with version 2, so these are Angular 2 notes.

## Upgrading Angular Version

To upgrade the versions of npm packages

```bash
$ npm update -D && npm update -S
```

To upgrade the version of angular CLI to the latest release

```bash
$ npm uninstall -g @angular/cli
$ npm install -g @angular/cli@latest
$ npm install --save-dev @angular/cli
```

To upgrade TypeScript to the latest release

```bash
$ npm install -g typescript@latest
```

## Expressions

You can render a value from your component in your template with double curly braces.

```typescript
@Component({
  selector: 'foo',
  template: `
    <h1>{{ title }}</h1>
  `
})
export class AppComponent {
  title: string = 'Foo';

  constructor() {}
}
```

This will place the value of title ("Foo") in the `<h1>` tag.

The code in the curly braces is interpolated JavaScript, so you can manipulate values.  For example, you can add:

```typescript
@Component({
  selector: 'foo',
  template: `
    <h1>{{ numberOne + numberTwo }}</h1>
  `
})
export class AppComponent {
  numberOne: number = 1;
  numberTwo: number = 2;

  constructor() {}
}
```

This will place the number 3 in the `<h1>` tag.

## Property binding

### One way data flow

You use square brackets `[]` to wrap a property you want to set to a value in your component class.  For example:

```typescript
@Component({
  selector: 'foo',
  template: `
    <img [src]='logoUrl' />
  `
})
export class AppComponent {
  logoUrl: string = './img/logo.svg';

  constructor() {}
}
```

This will bind the `src` attribute value of the image in the template to the component class's `logoUrl` property.

However, changing the value of an attribute bound like this after it has been rendered will not update the value in the component class.  For example, if we bind to an `<input>` value and then update the input's value in the browser by typing into it, it will not update the value in the component class:

```typescript
@Component({
  selector: 'foo',
  template: `
    <input [value]='name' />
    {{ name }}
  `
})
export class AppComponent {
  name: string = 'Elliot';

  constructor() {}
}
```

### Two way binding

In order for the data to flow both ways (from the component class to the template and from the template to the component class), you need to use two way binding.

We can do this manually by using one way property binding and event binding:

```typescript
@Component({
  selector: 'foo',
  template: `
    <input [value]='name' (input)='updateName($event)' />
    {{ name }}
  `
})
export class AppComponent {
  name: string = 'Elliot';

  constructor() {}

  updateName(event) {
    this.name = event.target.value;
  }
}
```

Angular makes this a bit easier with the `FormsModule`, allowing you to use the `ngModel` attribute on input fields:

```typescript
@Component({
  selector: 'foo',
  template: `
    <input [(ngModel)]='name' />
    {{ name }}
  `
})
export class AppComponent {
  name: string = 'Elliot';

  constructor() {}
}
```

## Event binding

You can use a binding approach similar to property binding to bind to events.  By wrapping an event handler attribute in parens `()` and assigning the attribute to the name of a method in your component class, that method will be called when the event is triggered.  For example, lets log out some text when clicking a link:

```typescript
@Component({
  selector: 'foo',
  template: `
    <a href='#' (click)='sayFoo($event)' />
  `
})
export class AppComponent {
  constructor() {}

  sayFoo(event) {
    event.preventDefault();
    console.log('foo foo foo');
  }
}
```

Notice in the method attribute we pass in a special argument `$event`, which allows us to call `preventDefault` on it.

## Template `#ref` variables

This allows you to reference a template element elsewhere in a template.  For example, lets pass the value of an input element to an event handler:

```typescript
@Component({
  selector: 'foo',
  template: `
    <input type='text' #myStatement />
    <a href='#' (click)='logIt($event, myStatement.value)'>Say it</a>
  `
})
export class AppComponent {
  logIt(event: any, statement: string) {
    console.log(statement);
  }
}
```

## Template directives

### `ngIf`

You can choose to conditionally show or hide an element in the template with the `*ngIf` directive.  In this example the div containing "Hello..." will only be rendered if the lenght of the `name` property is at least one character long.

```typescript
@Component({
  selector: 'foo',
  template: `
    What's your name?
    <input type="text" [(ngModel)]="name" />
    <div *ngIf="name.length">
      Hello, {{ name }}.
    </div>
  `
})
export class AppComponent {
  name: string: '';
  constructor() {}
}
```

### `ngFor`

You can iterate over a collection in a template with an `*ngFor` directive:

```typescript
@Component({
  selector: 'foo',
  template: `
    <h2>Users</h2>
    <ul>
      <li *ngFor="let user of users; let i = index">
        {{ i + 1 }}
        {{ user.firstName }} {{ user.lastName }}:
        <a href="mailto:{{ user.email }}">{{ user.email }}</a>
      </li>
    </ul>
  `
})
export class AppComponent {
  users: User[] = [
    {
      id: 1,
      firstName: 'Elliot',
      lastName: 'Larson',
      email: 'elliot@onehouse.net'
    },
    {
      id: 2,
      firstName: 'Ricky',
      lastName: 'Ahn',
      email: 'ricky@onehouse.net'
    },
    {
      id: 3,
      firstName: 'Nolan',
      lastName: 'Ehrstrom',
      email: 'nolan@onehouse.net'
    }
  ];
  constructor() {}
}
```

Notice the use of the `i` variable.  We reference this to the `index` variable made available to us by angular in the loop.

### Class binding and `ngClass`

You can set a class of an element based on a boolean value, like this:

```typescript
@Component({
  selector: 'foo',
  style: `
    .wears-a-sombrero {
      background: url('../img/sombrero.png') no-repeat;
      background-size: 24px 24px;
      padding-left: 30px;
    }
  `,
  template: `
    <h2>Users:</h2>
    <ul>
      <li *ngFor="let user of users">
        <div [class.wears-a-sombrero]='user.likesMexicanFood'>
          {{ user.firstName }} {{ user.lastName }}:
          <a href="mailto:{{ user.email }}">{{ user.email }}</a>
        </div>
      </li>
    </ul>
  `
})
export class AppComponent {
  users: User[] = [
    {
      id: 1,
      firstName: 'Elliot',
      lastName: 'Larson',
      email: 'elliot@onehouse.net',
      likesMexicanFood: true,
    },
    {
      id: 4,
      firstName: 'Arum',
      lastName: 'Ahn',
      email: 'arum@onehouse.net',
      likesMexicanFood: false,
    },
    {
      id: 2,
      firstName: 'Ricky',
      lastName: 'Ahn',
      email: 'ricky@onehouse.net',
      likesMexicanFood: true,
    },
    {
      id: 3,
      firstName: 'Nolan',
      lastName: 'Ehrstrom',
      email: 'nolan@onehouse.net'
      likesMexicanFood: true,
    }
  ];
  constructor() {}
}
```

You can do the same thing with `ngClass`.  The syntax is a bit more long-winded, however it allows you to add additional class conditionals to the same statement.

```html
<h2>Users:</h2>
<ul>
  <li *ngFor="let user of users">
    <div [ngClass]='{ "wears-a-sombrero": user.likesMexicanFood, "wears-a-kasa": user.likesSushi }'>
      {{ user.firstName }} {{ user.lastName }}:
      <a href="mailto:{{ user.email }}">{{ user.email }}</a>
    </div>
  </li>
</ul>
```

### `ngStyle`

This works in almost the same way as `ngClass`.  You can either use the `[style.background]` approach, or the `[ngStyle]="{ background: 'red', color: 'white' }"`.

## Pipes

You can modify values for they get interpolated with pipes.  For example, if you were to try to output an object in a template, you would get `[object Object]` as the output.  But you can use the `json` pipe to get the desired output:

```html
<h2>Users:</h2>
<ul>
  <li *ngFor="let user of users">
    {{ user | json }}
  </li>
</ul>
```

You can also chain pipes:

```html
{{ todo.dueDate | date: 'yMMMMd' | uppercase }}
```

[Here is a list of built in Angular pipes.](https://angular.io/docs/ts/latest/api/#!?query=pipe)

## Component architecture

Generally you have a parent component with data, that gets passed down into nested, child components.  The data in these child components can be bubbled up using events.

### Passing data to child components with `Input`

To send data to a child component, you create and `Input` on the child class and you pass it using a bound attribute on the child component directive used in the parent component's template:

```typescript
@Component({
  selector: 'parent',
  template: '<child [user]="users[0]"></child>',
})
export class ParentComponent {
  users: User[];

  ngOnInit() {
    this.users = [
      {
        firstName: "Jeffrey",
        lastName: "Lebowski"
      }
    ]
  }
}

@Component({
  selector: 'child',
  template: `
    <div>{{ user.firstName }}</div>
    <div>{{ user.lastName }}</div>
  `
})
export class ChildComponent {
  @Input() user: User;
}
```

If you are iterating over the array of users in the parent template, you may use the `ngFor` approach.  Notice how we can iterate over the users with the `ngFor` loop, and that we have access to the current iteration's `user`, which we bind to the input attribute for the child component.

```typescript
@Component({
  selector: 'parent',
  template: '<child *ngFor="let user of users" [user]="user"></child>',
})
export class ParentComponent {
  users: User[];

  ngOnInit() {
    this.users = [
      {
        firstName: "Jeffrey",
        lastName: "Lebowski"
      }
    ]
  }
}
```

### Getting data back out of a child component with `Output`s

When you want to pass data back up to a parent component, you use an output attribute on the child component and you bind to a custom event on the child component's directive.  You then `emit` the custom event with an `EventEmitter`.

```typescript
@Component({
  selector: 'parent',
  template: '<child [user]="users[0]" (update)="handleUpdate($event)"></child>',
})
export class ParentComponent {
  users: User[];

  ngOnInit() {
    this.users = [
      {
        id: 1,
        firstName: "Jeffrey",
        lastName: "Lebowski"
      }
    ]
  }

  handleUpdate(event: User) {
    this.users = this.users.map((user: User) => {
      if (user.id == event.id) {
        user = Object.assigns({}, user, event);
      }
      return user;
    });
  }
}

@Component({
  selector: 'child',
  template: `
    {{ user.firstName }} {{ user.lastName }}
    <input type="text" [(ngModel)]="user.firstName">
    <input type="text" [(ngModel)]="user.lastName">
    <button (click)="updateClicked()">Update</button>
  `
})
export class ChildComponent {
  @Input() user: User;
  @Output() update: EventEmitter<User> = new EventEmitter();

  updateClicked() {
    this.update.emit(this.user);
  }
}
```

## Angular CLI

### Testing a single file

It looks like the `ng test` command doesn't give you the ability to run specs for a single file, but you can do it with `karma` directly:

First, start the `karma` test runner with `start`.  This will unfortunately run all of your tests when it starts up:

```bash
$ node_modules/.bin/karma start
```

Then you can run the tests for a single file by calling the `karma run` command with the `--grep` option.

The string provided for grepping is the top level describe block for the test file you care about.

```bash
$ node_modules/.bin/karma run -- --grep='Todo model'
```

This will only run the test for that file once.  If you want it to run each time the test or implementation file are updated, you can use `nodemon`:

```bash
$ node_modules/.bin/nodemon -x "node_modules/.bin/karma run -- --grep='Todo model'" -w src/app/models/todo.model.ts -w src/
app/models/todo.model.spec.ts
```

## Dependency injection

Angular includes a dependency injection framework.  The use of DI in Angular allows you to more easily test classes with dependencies.  Instead of placing dependencies inside of a class, we pass them into the constructor (inject them).  This is what's referred to as composition.  We compose a class by passing dependencies into it.  In Angular you define the dependencies you want your class to have in the constructor.  For example, we might have a `Car` class that uses an instance of an `Engine` class.  The constructor might look like `constructor(private engine: Engine)`.  Then we might instantitate it with `new Car(new Engine())`.  So, the engine dependency is passed into the `Car` class.  However, in Angular the dependency injection is handled by the DI framework.  The `Car` instance is created by the framework, and it handles creating an instance of the `Engine` class and passing into the constructor of the `Car` class for you.

In order for a class to be available as an injectable, you need to add it either to either a module or component providers section.

For example, here lets make a TodoService available application wide by adding it to the `providers` array of the `app.module.ts` file:

```typescript
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    FirebaseModule,
  ],
  providers: [
    TodoService
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Now, when you create a component with the `TodoService` in the constructor, Angular will inject it for you when creating an instance.

### `useClass`

Then, when you're testing a component that uses this service, you can tell the injector to use a mock in the testbed, like so:

```typescript
TestBed.configureTestingModule({
  declarations: [ TodosComponent ],
  providers: [
    { provide: TodoService, useClass: MockTodoService }
  ]
})
.compileComponents();
```

Instead of passing the `TodoService` directly to the `providers` array of the `TestBed`, we pass an object with two keys.  The `provide` key is set to the expected service class, and the `useClass` key is set to the mock used for testing.

### `useValue`

You can also use the `useValue` key instead of the `useClass` key.  `useClass` passes in the class you want to instantiate using the DI system.  If you use `useValue` instead, this becomes the value used as the instance, instead of instantiating an instance of the class.  So, if you want to instantiate your own instance outside the DI system and then just pass this in, this is your option.

### `useFactory`

If you need to define a custom factory method for instantiating an instance, you can use `useFactory`.  Here you pass in a method that constructs the instance.  You also need to provide a `deps` array that defines the objects that need to be instantiated and passed into the method.

```typescript
let heroServiceFactory = (logger: Logger, userService: UserService) => {
  return new HeroService(logger, userService.user.isAuthorized);
};

export let heroServiceProvider =
  { provide: HeroService,
    useFactory: heroServiceFactory,
    deps: [Logger, UserService]
  };
```

Then, elsewhere in the providers array:

```typescript
{
  providers: [heroServiceProvider]
}
```

## Testing

### `TestBed`

The `TestBed` helps to create an isolated testing environment for testing components.  It is essentially a special `NgModule` that you attach your component to.  The `TestBed` class's `configureTestingModule` takes a hash like `NgModule`.


