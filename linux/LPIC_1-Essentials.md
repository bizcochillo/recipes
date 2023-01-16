# 03 - INSTALLING CENTOS 7

## Configuring network
To show the different network cards

```console
nmcli conn show
```

Bring interfaces up

```console
nmcli conn up <name>
```

Address where configure network when booting: `/etc/sysconfig/network-scripts/ifcfg-<interface_name, i.e. enp0s3>`. 

To enable an specific interface at boot time:

```console
sed -i s/ONBOOT=no/ONBOOT=yes/ /etc/sysconfig/network-scripts/ifcfg-enp0s8
```

## Installing X

Update software

```console
sudo yum udpate
```

Add other utilities

```console
yum install -y redhat-lsb-core net-tools epel-release kernel-headers kernel-devel
```

To install group of packages. 

```console
yum groupinstall -y "Development Tools"
```

```console
yum groupinstall -y "X Windows System" "MATE Desktop" 
```

To set graphical environment 

```console
systemctl set-default graphical.target
```

```console
systemctl isolate graphical.target
```
 
# 04 - WORKING AT THE COMMAND LINE
## Welcome to the command line
  
Physical TTY (real console) 

Local Pseudo TTY (graphical environment console) 

Remote Pseudo TTY (SSH) 

## Accessing consoles

`Ctrl + Alt + F1` (Main graphical console) 

`Ctrl + Alt + F2-F6` (Physical console)

To show which console you are log on

```console
tty
```

To show who is logged on on the system

```console
who
```

## Listing files

To show how aliased is a command

```console
type ls
```

To show all files

```console
ls -a
```

To show all files with a / at the end for directories (light blue for links and with -F it gets a decorator with @ at the end of filename). 

```console
ls -aF
```

For showing reverse order add a r. If the r is added and it's followed by a -t it shows the most recent file added sorted. 

```console
ls -lrt
```

With -h for the human readable format. 

```console
ls -lhrt
```

To show only the directory (Modificator `d`)

```console
ls -ld /etc
```

## File types

To show the devices on the system

```console
lsblk
```

To list the files which represents partitions or devices

```console
ls -l /dev/sda
ls -l /dev/sda*
ls -l /dev/sda?
ls -l /dev/sda[12]
```

To show different files with one ls 

```console
ls -l /etc/system-release /etc/centos-release /etc/redhat-release
```

To show the content of the release in CentOS

```console 
cat /etc/redhat-release
lsb_release -d
```

To query the install database

```console
rpm -qf /usr/bin/lsb_release
rpm -qf $(which lsb_release)
```

## Working with files

To enable interactive mode by copying, add -i modifier

```console
cp -i /etc/hosts .
```

## Working with Directories

To create a directory with the parent 

```console
mkdir -p sales/test 
```

Delete directory with rmdir

```console
rmdir test
```

Last command with filtering

```console
!rm
```

To remove recursively and the files 

```console
rm -rf sales
```

To create several directories with one command

```console
mkdir one two
```

To create a several files
```console
touch one/file{1..5}
```

To copy one directory and all its content to another
```console
cp -R one two
```

To add permissions (read-write-execution) to user/group/others we need to add the m modifier when creating a folder
```console
mkdir -m 777 d1
mkdir -m 700 d2 
```

## Working with links

Hard link count in the ls -l. To create a link

```console
ln f1 f2
```

to print the index number of a file or directory add a modifier -i 

```console
ls -li <dir>
```

 With symbolic links we can access accross file system. However with hard links we can share data in the same file system

```console
ln -s /boot/vmlinuz-3<etc> . [ALLOWED]
ln /boot/vmlinuz-3<etc> . [NOT ALLOWED]
```

# 05 - READING FILES

## Reading from files

To show data about the SSH connection

```console
echo $SSH_CONNECTION 
```

To cat two files

```console
cat /etc/hosts /etc/hostname
```

To count lines

```console
wc -l /etc/services
```

To see files (last argument) (`/` for searching forward) and `?` for searching for backwards)

```console
less !$ 
```

To see top 10 lines

```console
head -n 10 /etc/services
```

To see top n lines

```console
head -n <number of lines> /etc/services
```

To see last 10 lines

```console
tail /etc/services
```

To see last n lines

```console
tail -n <number of lines> /etc/services
```

## Regular Expressions and grep

To filter with pipe

```console
yum list installed | grep kernel
yum list installed | grep ^kernel
```

Install network time protocol

```console
sudo yum install ntp 
```

To match with a whole workd 'server'

```console
grep '\bserver\b' ntp.conf
```

## Using sed to Edit files

To see output when sed is executed

```console
sed '/^#/d ; /^$/d' ntp.conf
```

To actually edit the file (add parameter -i)

```console
sed -i '/^#/d ; /^$/d' ntp.conf 
```

## Comparing files

To compare two files

```console
diff ntp.conf ntp.new
```

To compare binary files

```console
rpm -V ntp
```

To compare binary files we can compare checksum

```console
md5sum /usr/bin/passwd
```

## Finding files

To find files in a directory using file name

```console
find /usr/share/doc -name '*.pdf' 
```

The default action is to print in the screen. 

```console
find /usr/share/doc -name '*.pdf' -print
```

To execute a command with the retrieved file. 

```console
find /usr/share/doc -name '*.pdf' -exec {} . \;
```

To delete de found files

```console
find -name '*.pdf' -delete
```

To find the symbolic links

```console
find /etc -type l
```

To limit the depth of the search 

```console
find /etc -maxdepth 1 -type l
```

To find files larger than 20M 

```console
find /boot -size +20000k -type f
```

To see the disk size of files which are larger than 10M

```console
find /boot -size +10000k -type f -exec du -h {} \;
```

# 06 - USING THE VIM TEXT EDITOR

## Creating and editing files in Linux and using touch

To create a new file

```console
touch newfile
```

We can use redirecting

```console
>newfile1 
```

To see statistics about the file

```console
stat newfile
```

To change the date of the file 

```console
touch -d '10 April 1973' newfile
```

## Using the nano Text Editor

nano not part of the minimal install

```console
sudo yum install -y nano
```

## Learning vim with vimtutor

```console
vimtutor
```

To see the vim help: `:help`

## Editing files with vim

add .vimrc: `set showmode nonumber nohlsearch`

autoindent: `:set ai`

tab separator: `:set ts=4`

spaces instead of tab: `:set expandtab`

Set macro => nmap <C-N> `:set invnumber<CR>`
 
abbreviation=> abbr _sh `#!/bin/bash`
 
Revert to the last saved changed => `:e!`
 
Editing mode(insert and append) => `iIaA`
 
Copy lines: `yy` `2yy`
 
Paste lines: `p`
 
# 07 - PIPING AND REDIRECTION

## Redirecting STDOUT
 
to redirect explicitly the stdout  

```console 
df -h 1> file1
``` 

 Using the noclobber option for protecting overwriting. To see shell options

```console
set -o 
```
 
To turn noclobber on
 
```console
set -o noclobber
```
  
I can overwrite by adding a safety bar

```console
date +%F >| file1
```
 
It can be added to home directory  (vi .bashrc). To disable noclobber again

```console
set +o noclobber
```
 
## Redirecting STDERR

To redirect standard error

```console
ls /etcw 2>| err
```
 
To avoid permission denied messages clutter the output

```console
find /etc -type l 2> /dev/null
```
 
Standard output and standard error

```console
find /etc -type l &> err.txt
```
 
## Redirecting into STDIN

To see the file system information and send an mail

```console
df -hlT > diskfree
main -s "Disk Free" < diskfree
```
## Using HERE documents

To write in a file until I find the word END

```console
[tux@server1 ~]$ cat > mynewfile <<END
> this is a little
> file that we can crete
> even with scripts
> END
[tux@server1 ~]$ cat mynewfile
this is a little
file that we can crete
even with scripts
```

## Command pipelines

Command cut to extract delimiters from a file 
 
```console
[tux@server1 ~]$ head -n1 /etc/passwd
root:x:0:0:root:/root:/bin/bash
[tux@server1 ~]$ cut -f7 -d: /etc/passwd
/bin/bash
/sbin/nologin
```
 
Sort & unique commands

```console
cut -f7 -d: /etc/passwd | sort | uniq
```
 
## Named pipes

To create a pipe

```console
mkfifo mypipe
ls -F mypipe
(from one terminal) ls > mypipe
(from the other) wc -l < mypipe
```
                               
## Using the command tee

To redirect output to file and screen

```console
ls | tee f89
```

Hand over sudo command append with tee

```console
sudo echo '127.0.0.1 bob' | sudo tee -a /etc/hosts
```

# 08 - ARCHIVING FILES

## Using the tar commands

To create a tar file 

```console
tar -cf doc.tar /usr/share/doc
```

To add a verbose switch

```console
tar -cvf doc.tar /usr/share/doc
```
                               
To add even more verbosity

```console 
tar -cvvf doc.tar /usr/share/doc
```

To list the files that are in an archive. 
```console
tar --list --file=doc.tar
```

Same output

```console
tar -tf doc.tar
```

To extract a file in the current location

```console
tar --extract --verbose -f doc.tar
```
                               
In a directory

```console
tar --extract --verbose -f doc.tar --directory=/
```

Incremental backup: only the files that have changed since last backup. To create a tar snapshot and perform incremental backups

```console
tar -cvf my0.tar -g my.snar test
```

Add some new content to one existing files in the snapshot and createa new one

```console
echo hi >> test/hosts
tar -cvf my1.tar -g my.snar test
```

Remove files and create a new snapshot

```console
rm test/hostname
tar -cvf my2.tar -g my.snar test
```

Extract different snapshots

```console
tar -xvf my0.tar -g /dev/null
tar -xvf my1.tar -g /dev/null
tar -xvf my2.tar -g /dev/null
```

## Using compression
To compress a tar file (and remove it) 

```console
gzip tux.tar
```

To see the file type

```console
file tux.tar.gzip
```

To unzip the file. 

```console
gunzip tux.tar.gzip
```

To zip with a better algorithm

```console
bzip2 tux.tar
```

To see which format the file has. 

```console
file tux.tar.bz2
```

To unzip 

```console
bunzip2 tux.tar.bz2
```

To compress and add time

```console
(for normal archiving) time tar -cvf tux.tar $HOME
(for zip)  time tar -cvzf tux.tar.gz $HOME (add z) 
(for bzip2) time tar -cvjf tux.tar.bz2 $HOME (add j)
```

## Working with cpio

To compress

```console
find -name '*.pdf' | cpio -o > /tmp/pdf.cpio
```

To expand

```console
cpio -id < /tmp/pdf.cpio
```
                        
Also for compressing. It's usually the format for the image boot

## Imaging with dd
To create an image

```console
 dd if=/dev/sr0 of=netinstall.iso
```
 
Copy the first block of the MBR

```console
dd if=/dev/sda of=sda.mbr count=1 bs=512
```
# 09 - ACCESSING COMMAND LINE HELP

## Working with man pages

man section numbers 

```console
man man
       1   Executable programs or shell commands
       2   System calls (functions provided by the kernel)
       3   Library calls (functions within program libraries)
       4   Special files (usually found in /dev)
       5   File formats and conventions eg /etc/passwd
       6   Games
       7   Miscellaneous  (including  macro  packages  and  conventions), e.g.
           man(7), groff(7)
       8   System administration commands (usually only for root)
       9   Kernel routines [Non standard]
```

```console
man crontab
```

File format for crontab

```console
man 5 crontab
```

## Working with info pages

```console
info ls
 ```

## Using version control systems

We can check in the version (creates a ,v file)

```console
ci hello.sh
```
 
To see the version

```console
rlog -b hello.sh
```

To checkout the file (-l for lock). End comment with dot (.)

```console
co -l hello.sh
```

To checkout a specific version
```console
co -l -r1.1 hello.sh
```
To revert back to that version

```console
ci -f hello.sh
```

# 10 - UNDERSTANDING FILE PERMISSIONS

## Listing permissions

To show the file system type

```console
df -hT
```

To see the filesystem mounted we can use mount

```console
mount | grep root
```

Stat command for listing permissions. (User, Group and Others)

```console
stat hello.sh,v
```

To extract the simbolic permissions

```console
stat -c %A hello.sh,v
```

To extract the octal description

```console
stat -c %a hello.sh,v
```
 
## Managing Default permissions

Mask by default by creating a file

```console
umask
```
 
Set writable for all groups

```console
umask 0
```
 

Set only to groups
 
```console
umask 27 
```
 Set only to user

```console
umask 77
```
 
Equivalent for human readable umask (rw-r--r--.)

```console
umask u=rwx,g=rx,o=rx 
```

Set groups and others to nothing (Equivalent to 0077) 

```console
umask u=rwx,go=
``` 
## Setting Permissions

To set permissions with chmod

```console
chmod 467 file1
chmod u=r,g=rw,o=rwx file1
```console
 
To add execution permission to a file

```console
chmod a+x file2
```

To remove permission from a file

```console
chmod a-x file2
```

We can group permissions

```console
chmod go= file2
```

The fourth group includes the sticky bit, the group id bit and the userid bit. Sticky bit controls deletions from the directory (t => execution permissions is set, T=> not set) 

```console
ls -ld /tmp
```

To create a directory with the sticky bit

```console
mkdir -m 1777 test
```

 To remove the sticky bit

```console
chmod o-t test
```
 
Group ID bit (s). (Wall is set message to users, to terminals) When the command runs, it runs with the permission of the group

```console 
ls -l $(which wall)
```
 
 User Id bit set (for instance). It executes as root, enabling users to change the their own password

```console
 > ls -l $(which passwd)
```
 
## Managing File ownership

To see the id

```console
id -u
```

To see the id (name)

```console
id -un
```
 
To see the group (name) primary group

```console
id -gn
```
  
 Complementary group 

```console
id -Gn
```
 
To change the group of a file

```console
chgrp wheel file2
```
 
To change the primary group

```console
newgrp wheel
```
 
(Primary group: create resources. Secondary group: accessing resources). To change the owner

```console
chown root file2
```
 
 To change user and group

```console 
chown tux.tux file2
```
 
Copy file changing ownership of user who copies 

```console 
cp file2 /root/file2na
```
 
 Copy file keeping original ownership

```console
cp -a file2 /root/file2a
```
 
# 11 - ACCESSING THE ROOT ACCOUNT

## Using su

```console
su
id
```
 
Creates a new bash shell and stays with the current user.

```console 
echo $USER
```
 
 Dollar is a sign that the logged user is regular user. To add full access with the account context root

```console
su -l 
```
 
## Implementing sudo

Five minuts period is keep the auth. To see the sudo parameters

```console 
 > sudo visudo
```
 
User_Alias and Command_Alias for host and commands allowed. For showing the options

```console
 > sudo sudo -V 
```
 
## Restricting root access to SSH

To access as root

```console
ssh -l root 192.168.56.102
```
 
 To edit the options for ssh connection

```console 
vi /etc/ssh/sshd_config
(Edit PermitRootAccess to no)
```
 
restart the SSH daemon

```console
systemctl restart ssh
```
 
# 12 - ACCESSING SERVERS WITH SSH

## Configuring SSH client

For the known hosts: `cat .ssh/known_hosts`
config the ssh access based on alias on config file into `~/.ssh/config` file

## Using key based auth

To generate a key

```console
ssh-keygen -t rsa
```
 
To copy the key to the server 

```console
ssh-copy-id -i id_rsa.pub server2
ssh server2
```
 
And the target machine has the keys on 

```console
cat .ssh/authorized_keys
```
 
Configure password restriction (PermitRootAccess) at `/etc/ssh/sshd_config`

## Copy files securely

```console
scp /etc/hosts server2:/tmp
```
 
In the opposite direction

```console
scp server2:/tmp/hosts .
```

# 13 - USING SCREEN AND SCRIPT

## Using script as collaboration

To records the session
 
```console
script
...

exit
```
 
Script to another window

```console
mkpipe /tmp/mypipe
script -f /tmp/mypipe
(other machine) cat /tmp/mypipe
```
 
## Running screen

```console
yum install -y screen
```

Needs ssh setup
