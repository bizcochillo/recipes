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

## System login scripts

## Home directory templates

# 03 - CREATING AND MANAGING LOCAL USERS IN CENTOS7

# 04 - MANAGING LOCAL GROUPS IN CENTOS7

# 05 - USING PAM TO CONTROL USER ACCESS

# 06 - IMPLEMENTING OPENLDAP DIRECTORIES ON CENTOS7

# 07 - IMPLEMENTING OPENLDAP AUTHENTICATION ON CENTOS7

08 - IMPLEMENTING KERBEROS AUTHENTICATION