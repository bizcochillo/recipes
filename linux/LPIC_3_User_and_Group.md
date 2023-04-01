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

# 03 - CREATING AND MANAGING LOCAL USERS IN CentOS7

# 04 - MANAGING LOCAL GROUPS IN CentOS7

# 05 - USING PAM TO CONTROL USER ACCESS

# 06 - IMPLEMENTING OpenLDAP DIRECTORIES ON CentOS7

# 07 - IMPLEMENTING OpenLDAP AUTHENTICATION ON CentOS7

# 08 - IMPLEMENTING KERBEROS AUTHENTICATION
