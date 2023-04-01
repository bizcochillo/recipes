# 01 - INTRODUCTION

## Start

PAM: Pluggable Authentication Modules

## Understanding getent command

See users with 

```console
 cat /etc/passwd
```

The same with getent

```console
getent
```

Where to look for other locations for password database

```console
grep passwd /etc/nsswitch.conf
```

To look for local groups:

```console
getent group
```

To look for networks

```console
getent networks
```

Same for services

```console
getent services
```

# 02 - MANAGING LOGING SCRIPTS

## Login Scripts with CentOS 7

## Investigating the Execution Order

To switch to the root account

```console
su -l
```

By issuing `exit` we execute the logout script bash exit. To assume the root credentials, we issue: 

```console
su -l
```

And we keep the environment variables for the user. For loading profiles, first is .bashrc and for profile specific stuff .bash_profile is loaded afterwards. With `su` (without arguments) it gets loaded only .bashrc but for `su -` or `su -l` both profiles are loaded.

## System login scripts

In the /etc/ folder there are a file `profile` and a directory `profile.d`. We usually don't update the profile file, intead we add a script to the profile.d directory

Variable `$PS1` contains prompt information about the user. If we want to change the prompt info for the users, edit the file `/etc/bashrc` file. 

## Home directory templates

`/etc/skel`: Template directory for creating user home directory (affect home directory of new users). Linux `source` command is to execute a command in a file. It's equivalent to . (dot)

Login: `/etc/profile`, `~/bash_profile`, `~/bashrc`, `/etc/bashrc`.

Non-login: `~/bashrc`, `/etc/bashrc`

Profile: 
- `PATH=$PATH:~/bin`
- `export PATH`

Bashrc
- `PS1="[\u@\h \w]\$"`

# 03 - CREATING AND MANAGING LOCAL USERS IN CentOS7

## Introduction

To identify user accounts issue `id`. Other users `id <user>`. To show the primary group `id -g`. To show also secondary groups issue `id -G` and also to show the name we add n: `id -Gn`

## Creating User Accounts

The command `useradd` is for creating local users in the system. The switch `-m` adds a home directory and without more arguments it uses the defaults. Therefore

To create the user `user1`: 

```console
sudo useradd -m user1
```

To check the local users (last added) in the local system: 

```
tail -n 1 /etc/passwd
```

For creating an user not belonging to a private group we use the switches -N <user> -g <public group>. For example, for creating an user belonging to the public group users and to the secondary group adm, we issue
 
```console
sudo useradd -N user2 -g users -G adm
```
 
We can create a third user `user3` with a private group, belonging to the secondary group administrator and with the `s` shell. 
 
```console
sudo useradd user3 -G adm -s /bin/sh
```

For checking Debian-like user scripts, we notice than in CentOS systems, the script `adduser` is a symbolic link to `useradd` instead the Debian one for creating user accounts. Check `ls -l /sbin/adduser`.

## Managing User Passwords

## Password Age Data

## Account Defaults

## Modify and Delete Accounts

# 04 - MANAGING LOCAL GROUPS IN CentOS7

# 05 - USING PAM TO CONTROL USER ACCESS

# 06 - IMPLEMENTING OpenLDAP DIRECTORIES ON CentOS7

# 07 - IMPLEMENTING OpenLDAP AUTHENTICATION ON CentOS7

# 08 - IMPLEMENTING KERBEROS AUTHENTICATION
