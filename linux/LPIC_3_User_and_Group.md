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

Change the default timeout for sudo, we issue `sudo visudo` and set the line `Defaults        timestamp_timeout=<here_your_timeout>`
 
To see password hashed in the system on the file `/etc/shadow`. To change the password on linux system, use chpasswd with the standard input to enable, for instance, batch user creation.
 
```console
 echo 'user2:Password1' | sudo chpasswd
 ```

 For RH systems we can use the `--stdin` flag with the `passwd` command as:  
 
 ```console
 echo 'user2:Password1' | sudo passwd user3 --stdin
 ```
 
## Password Age Data
 
In the `/etc/shadow` we see the aging information
 
To change the password expiry information, we use the command `chage`, for instance, for listing the info:
 
```console
chage -l tux
```

For not storing the password in the shadow file, but in the passwd file instead, issue the command `pwunconv`:

```console
sudo pwunconv
```

All users can read the file. With the shadow file, it can keep private to users and only for the root user. To convert it back to shadow data, issue: 

```console
sudo pwconv
```
 
To change the expiry time of the password of an user, issue: 

```console
sudo chage -M 40 user1
```
 
Locking an account, issue the command

```console
sudo passwd -l user1
```
 
By inspecting the shadow file, we can observe that the exclamation before the password hash is now present. 
 
To unlock the account
 
```console
sudo passwd -u user1
```

## Account Defaults

To see the defaults file inspect the file `/etc/login.defs`. 
 
PAM stands for Pluggable Authentication Module. To show the defaults, we also can issue the command `sudo useradd -D`. If we want to change the default shell for new users, we issue the command: 
 
```console
sudo useradd -Ds /bin/sh
```
 
The defaults file is located at `/etc/default/useradd`. Therefore, defaults can be changed directly by editing this file. 

## Modify and Delete Accounts

For instance to modify user to add the full name, we issue:
 
```console
sudo usermod -C "User One" user1
```

For listing the available shells: 

```console
chsh -l
```

For change my own shell: `chsh -s /bin/sh tux`. We can check it at `grep tux /etc/passwd`. To change again with usermod we issue `sudo usermod -s /bin/bash`. We issue it with the modifier `-s`.
 
For deleting an user, we issue the command 
 
```console
sudo userdel -r user1
```
 
The `-r` modifier will take care of the deletion of user home directory, mail spool and cron jobs. It does not search in the file system for files owned by the deleted user. Once a user is deleted and the deletion has not been performed with the `-r` modifier, we can delete files via the user ID in the file system by scrapping it with find, like
 
```console
sudo find /home -uid 1002 -delete
```

# 04 - MANAGING LOCAL GROUPS IN CentOS7

## Creating local groups

We can read the local group file as: 
 
```console
grep tux /etc/group
```
 
To change the primary group to `wheel`, we issue the command -switching group id- `newgrp wheel`. In CentOS and RH system groups are private by default. When we create a new file, the group shown will be `wheel`
 
To create a new group, we use the command groupadd. 

```console
sudo groupadd sales
```

We have the password file for groups at `/etc/gshadow`. Generally, groups does not have password. We have the similar set of programs for managing groups `groupadd`, `groupmod` and `groupdel`  
 
## Manage Group Membership

We can modify the secondary group membership of the user with `usermod -G`, but we need to specify all the groups the user belongs to. For instance:
 
```console
sudo usermod -G sales,wheel tux
```

It's possible to add an user to a group with the command `gpasswd` with the modifier `-a`. This is not the most obvious thing to do: 
 
```console
sudo gpasswd -a tux sales
```
 
For adding more than one user with the command, we can specify a list: 
 
```console
sudo gpasswd -M tux,root sales
```

To see the actual list of group membership with id, it's mandatory to login again. 

## Making use of the SGID permission

Setting the Group ID bit. With the Apache server installed, we inspect the permissions and groups of the apache server (configuration DocumentRoot at `/etc/httpd/conf/httpd.conf`) and set to `/var/www/html` folder. To change the group membership recursive for the Apache folder we issue: 
 
```console
sudo chgrp -R apache /var/www
```
 
To remove other permissions: 
 
```console
sudo chmod -R o= /var/www
```

To enable the special flag permission on the group level, we modify the permissions g+s on the current folder to have new content be created under the owning group membership
 
```console
chmod g+s .
umask 027
``` 

## Group password
 
To enable a group to have password, we issue the command `newgrp adm`. We can have an user accessing to other permission set granted by group membership. 
 
# 05 - USING PAM TO CONTROL USER ACCESS
 
How to control user access to resources. 
 
## Automating home directory creation at user login

Some PAM configuration locations are in `/etc/pam.d/` folder. The programs used by PAM are in `/lib64/security/`. On `/etc/security` we can start to configure some of the modules. The three locations work together to provide secure access management. 
 
In RH we check that we have the oddjob module for PAM: `rpm -qa | grep oddjob`. We need to enable and start the oddjob daemon and the auth module: 
 
```console
sudo yum install oddjob
systemctl enable oddjobd
systemctl start oddjob
sudo authconfig --enablemkhomedir --update
```

## Impelementing password policies
 
To see which policy applies, see `cat /etc/pam.d/system-auth`. Check the file `/etc/security/pwquality.conf`. We can fine tune those files to increase the password health. To see the password score, issue `pwscore`. The score is between 0 and 100. 

## Restricting or limiting access to resources
 
File `/etc/security/limits.conf` to set limits on the system. The file is well self-documented. 
 
## Adding login time restrictions

Control access during the day, for instance. At the directory `/etc/pam.d` we can configure. By addin a line like 
 
```console
account   required    pam_time.so
```

This module reads then the configuration on the `/etc/security/time.conf` file, which is also well-documented. For instance, for not allowing access on workday between 8 and 18 for users bob and tux we add to the file the line:

```console
*;*;tux|bob;!Wk0800-1800
``` 
 
# 06 - IMPLEMENTING OpenLDAP DIRECTORIES ON CentOS7

The goal is to configure an LDAP Server to support centralized login
 
## Install OpenLDAP

We need to make sure that the fully qualified name gets resolved, so, hostname to ensure we have the domain configured and add the host ip to the host file like `echo "my_ip_address server1.example.com"`. Afterwards, we need to put the firewall exception and install openldap: 
 
```console
firewall-cmd --permanent --add-service=ldap
firewall-cmd --reload
yum install -y openldap openldap-clients openldap-servers migrationtools.noarch
```

With the migration tools, we can take the users present in the `/etc/passwd` file and create them in the open LDAP directory

## Configure OpenLDAP

First, we can pick the example configuration to put in the proper directory:
 
```console
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
```

We can check the configuration with `slaptest`. To check that the DB files are correctly set with the right owner (ldap user, ldap group) and enable/start the service, we issue:
 
```console
chown ldap.ldap /var/lib/ldap/*
systemctl enable slapd
systemctl start slapd
```

To add entries to the schema
 
```console
cd /etc/openldap/schema/
ldapadd -Y EXTERNAL -H ldap:/// -D "cn=config" -f cosine.ldif
ldapadd -Y EXTERNAL -H ldap:/// -D "cn=config" -f nis.ldif 
```
 
For changing the configuration, we create a config.ldif file with the following content: 

```console
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}hETVGKh8af1csLNzpCVfwhBX7n5yahbY

dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: 0

dn:olcDatabase={1}monitor,cn=config
changetype: modify
replace:olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=example,dc=com" read by * none
```

And issue the command 
 
```console
ldapmodify -Y EXTERNAL -H ldapi:/// -f config.ldif
``` 
 
## Create directory structure

The file structure.ldif created: 

```console
dn: dc=example,dc=com
dc: example
objectClass: top
objectClass: domain

dn: ou=people,dc=example,dc=com
ou: people
objectClass: top
objectClass: organizationalUnit
 
dn: ou=group,dc=example,dc=com
ou: group
objectClass: top
objectClass: organizationalUnit
```

And then we issue the command to create the objects: 
 
```console
ldapadd -x -W -D "cn=Manager,dc=example,dc=com" -f structure.ldif
```
 
To search the ldap directory: 
 
```console
ldapsearch -x -W -D "cn=Manager,dc=example,dc=com" -b "dc=example,dc=com" -s sub "(objectclass=organizationalUnit)"
```
 
## Create groups and users

We create a LDAP object upon the file group.ldif to host all LDAP users and then, we will use the migration tool. The `group.ldif` file:
 
```console
dn: cn=ldapusers,ou=group,dc=example,dc=com
objectClass: posixGroup
cn: ldapusers
gidNumber: 4000
```

To create the object:

```console
 ldapadd -x -W -D "cn=Manager,dc=example,dc=com" -f group.ldif
```

Where the migration tool is located is in the folder `/usr/share/migrationtools/`. There we edit the file `migrate_common.ph` (variables `DEFAULT_MAIL_DOMAIN` and `DEFAULT_BASE`).

When using the password migration, we can use the tux as template 
 
```console
grep tux  /etc/passwd > passwd
/usr/share/migrationtools/migrate_passwd.pl passwd user.ldif
vi user.ldif   #edit the file to comply with the LDAP directory standards
ldapadd -x -W -D "cn=Manager,dc=example,dc=com" -f user.ldif
```

# 07 - IMPLEMENTING OpenLDAP AUTHENTICATION ON CentOS7

# 08 - IMPLEMENTING KERBEROS AUTHENTICATION
