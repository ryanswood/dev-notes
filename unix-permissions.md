How to create a symlink:

Usage:
ln [OPTION]... [-T] TARGET LINK_NAME   (1st form)

Arguments:
-s, --symbolic              make symbolic links instead of hard links
-f, --force                 remove existing destination files

To create a new symlink (will fail if symlink exists already):
ln -s /path/to/file /path/to/symlink

To create or update a symlink:
ln -sf /path/to/file /path/to/symlink


Creating a user

Use cases
1. Create SSH key only user. No password


https://askubuntu.com/questions/345974/what-is-the-difference-between-adduser-and-useradd


Changing password
chpasswd - changes password non-interactively for one or more users
E.g. echo $user:$password | chpasswd

passwd - interactively change password for user

Arguments:
-d, --delete
           Delete a user's password (make it empty). This is a quick way to
           disable a password for an account. It will set the named account
           passwordless.


Changing the group for a file
chgrp <group_name> /path/to/dir


Change user and group ownership for dir/file
chown root:ablecustomer /path/to/dir


Chroot + SFTP only:

```
Match Group sftpusers
  ChrootDirectory /home/%u
  AuthorizedKeysFile /home/%u/.ssh/authorized_keys
  ForceCommand internal-sftp -l INFO

1. Put user in sftpuser group
2. create /home/user/<sftp>
3. Change ownership of users home directory to root
4. move their customer data symlink to /home/user/sftp

```

Resources: https://www.digitalocean.com/community/tutorials/how-to-enable-sftp-without-shell-access-on-ubuntu-16-04
https://stackoverflow.com/questions/8440537/is-it-possible-let-chroot-jails-share-directoriesread-only-outside-the-jail


Copying files (scp):

From local to remote:
`scp /path/to/local user@domain.com:~/remote/location/to/store`

From remote to local:
`scp user@domain.com:~/remote/location ./path/to/local`


List groups:

List groups for user (logged in as the user): `groups`
List groups for user : `groups <user>`
List all groups: `cut -d: -f1 /etc/group | sort`
