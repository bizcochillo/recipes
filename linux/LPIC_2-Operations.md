01 - INTRODUCTION
* Reading operating system data
$ cat /etc/system-release
$ lsb_release -d
To query the installation database
$ rpm -qf $(which lsb_release)
To see the kernel version
$ uname -r
Also an alternative
$ cat /proc/version
Information about boot
$ cat /proc/cmdline
Partition are separated 
$ lsblk

02 - STARTING AND STOPING CENTOS 7

* Introduction and using wall
$ write user
Create a file by editing. 
# cat > message <<END
Send message to all terminals
# wall < message 
To see status of messaging (does not affect root)
$ mesg
To disable
$ mesg n
To enable
$ mesg y

* Shutdown commands
Commands for going down 
$ poweroff
$ halt
$ reboot
Legacy commands
$ init --help
# telinit --help
To schedule and advise a shutdown
# shutdown -h 10 "The system goes home in 10 m"
To cancel a shutdown
# shutdown -c
When it remains less than 5 minutes a message is displayed at /run/nologin

* Changing runlevels
We can see the runlevel. (5 graphical number)
$ who -r 
With the runlevel command we get the same
# runlevel
To get the graphical environment
# systemctl get-default
To set the multiuser target(3)
$ systemctl set-default multi-user.target
systemctl isolate
* Selecting runlevels

03 - THE BOOT PROCESS
* Managing GRU Recovery
To edit grub file
# vi /etc/default/grub
GRUB_DISABLE_RECOVERY="false"
To generate grub configuration (add recovery mode for us)
# grub2-mkconfig -o /boot/grub2/grub.cfg
* Reset Lost Root Passwords (Boot to initramfs)
Edit linux16 entry
Remvoe rhgb and quiet add rb.break enforcing=0
switch_root# mount -o remount, rw /sysroot ; chroot /sysroot
sh-4.3$ mount -o remount,ro /sysroot; exit
# restorecon /etc/shadow
System continues to boot

04 - MANAGING GRUB2
* Introduction and re-installing GRUB
# grub2-install /dev/sda/
To install grub for a EFI based machine
# yum reinstall grub2-efi shim 
* Manage GRUB2 Defaults
# vi /etc/default/grub
# grub2-mkconfig -o /boot/grub2/grub.cfg
* Manage Persistent Settings with grubby
To see the default kernel
# grubby --default-kernel
To change to default kernel
# grubby --set-default /boot/vmlinuz-3.10.0-327.el7.x86_64
To see all GRUB info 
# grubby --info=ALL
To see only one kernel
# grubby --info= /boot/vmlinuz-3.10.0-327.el7.x86_64
To remove arguments from kernel initialization (!$ for last argument)
# grubby --remove-args="rhgb quiet" --update-kernel !$
* Password protect GRUB2
To create password protected grub entry go to edit /etc/grub.d/01_users
grub2-install /dev/sda
vi /etc/grub.d/01_users
To create password protected passwords
# grub2-mkpasswd-pbkdf2
* Custom GRUB2 entries
edit the 40_custom file at /etc/grub.d

05 - MANAGING LINUX PROCESSES
* Listing processes with ps
Show processes
# ps
Show all processes
# ps -e
Show all process and not associated with terminal
# ps aux
Show tree form
# ps -e --forest
To show better process tree
# pstree
To show all information
# ps -f
And even more information
# ps -F
Long listing for processes
# ps -l
Replace RSS for memory used
# ps -ly
To show everything again
# ps -elf
and filtering
# ps -elf | grep sshd
* Using the /proc directory and the $$ variable
# cd /proc
To see process one
# ps -p1 -f
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:18 ?        00:00:01 /usr/lib/systemd/systemd --switc
echo $$ gets the current PID of the current process
# ps -p $$ -F
Current working directory 
# ls -l cwd
Get the executable
# ls -l exe
The last process id run
# cat loadavg
* Manage processes with kill
To list all signals
# kill -l
For termination signal
# kill <pid>
# kill -15 <pid>
# kill -term <pid>
# kill -sigterm <pid>
# kill -sigkill <pid>
To list all keyboard shortcuts
# stty -a 
* Shortcuts with pgrep and pkill
For instance to filter the processes
# pgrep sshd
To use pgrep output
# ps -F -p $(pgrep sshd)
To start processes in the background, add & to the end
# sleep 100&
To kill all pids with process name sleep
$ pkill sleep
Command top for listing processes
press e for showing size in different units. f for fields and orderingq
* Monitor resource usage with top

06 - PROCESS PRIORITY
* Background tasks
$ sleep 1000&
To see jobs
$ jobs
Ctrl+Z stop the process
$ bg start the stopped job in the background
To bring to the foreground
$ fg 1 
When we kill the parent, the first process (pid 1) becomes the direct antecessor. 
$ ps -F -p $(pgrep sleep)
* Using nice to control process priority
Priority column CPU priority. It's controlled by the nice value. (nice from -20 to 19. The higher the number, the less CPU an app gets)
To set priority
$ nice -n 1 sleep 1000&
To put new priority
$ renice -n 10 -p 3403
To set limits
$ vi /etc/security/limits.conf

07 - MONITOR LINUX PERFORMANCE
* Listing standard tools in the proc tools next generation
$ rpm -ql procps-ng
To see to which package a tool belongs to
$ rpm -qf /usr/bin/top
Package -ql procps-ng
List documentation
$ rpm -qd procps-ng
List configuration
$ rpm -qc procps-ng
List only programs
$ rpm -ql procps-ng | grep '^/usr/bin/'
Number of programs
$ rpm -ql procps-ng | grep '^/usr/bin/' | wc -l
* Using pwdx and pmap
Free in megabytes
$ free -m
In gb
$ free -g
To see how memory maps to a running proccess (current) 
$ pmap $$
pwdx to see process working directory
$ pwdx $$
$ pwdx $(pgrep sshd)
* Working with uptime and tload
the uptime of the system. load average 1-5-15 mins
$ uptime
who is  logged 
$ who
who + uptime
$ w
List CPU information
$ lscpu 
cat /proc/uptime shows total uptime in seconds and idle time 
cat /proc/loadavg shows average usage
To every four seconds issue an uptime command
$ watch -n 4 uptime
tload is intended by design to show the average load
* recording performance with top and vmstat
To see one interation (batch mode)
$ top -b -n1
To see the size in k (display unit - kilobytes)
$ vmstat -S k
To collect iteration every five minutes three times (in megabytes)
$ vmstat -S m 5 3

08 - USING SYSSTAT TO MONITOR PERFORMANCE
To produce and deliver reports on system use 
* Installing sysstat
# yum install -y sysstat
Generate periodic report 
# cat /etc/cron.d/sysstat
Configuration of sysstat
# cat /etc/sysconfig/sysstat
Start, enable and query service sysstat
# systemctl start sysstat
# systemctl enable sysstat
# systemctl status sysstat
* Additional tools
See IO stats
$ iostat
in mb
$ iostat -m 5 3
Collect stats for an individual process
# pidstat -p $$ 5 3
Collect memory stats
# mpstat -P ALL 2 3
* Creating system activity reports
Log file at /var/log/sa/
# sar -u (CPU)
# sar -r (Memory)
# sar -q (Load averages)
# sar -n DEV (network)
Start and end period
# sar -s 14:00:00 -e 15:00:00
sar files
# sar -f /var/log/sa/sa15

09 - MANAGING SHARED LIBRARIES
* Viewing shared library modules
$ ldd /usr/bin/ls
/lib64/libpcre.so.1 is the posix common regular expressions
so extension => s standard o modules
Loaded 
$ pmap $$
list modules for bash
$ ldd /usr/bin/bash
* Sharing locations
lib directory is a link to /usr/lib and lib64 to /usr/lib64
* Sharing library cache
To show the location if we are in a symbolic link
$ pwd -P
Additional configuration files (extensions)
$ cd /etc/ld.so.conf.d
Independent locations where libraries can be load
LD_LIBRARY_PATH
Update library cache to use the modules
/etc/ld.so.cache
To be able to write the cache file
$ ldconfig -p
# ldconfig

10 - SCHEDULING TASKS IN LINUX
* Simple shell scripts
mail script
#!/usr/bin/bash - shebang (for starting the file)
FILE=/tmp/df.txt - variable
df -h > $FILE
mail -s "df" tux < $FILE && rm $FILE (&& makes that the second half executes only if the first part succeeds)
* Scheduling tasks with crond
Check if crond is running
# systemctl status crond
/etc/crontab
/etc/cron.d (within this file we can put files formated in the same way
Cron directories
$ ls /etc/cron*
To see the crontab
vi /etc/crontab
crontab -l
Edit
crontab -e
Remove completely 
crontab -r
Populate MAILTO to have output mailed to the user. 
For configuring cron:
minute hour day-of-month month day-of-week
* Using anacron as a Job Scheduler
Schedule jobs with a delay in minutes after the system starts
/etc/anacrontab
Format: period delay job-id command
* Irregular Job Scheduling with at
# systemctl status atd
# at noon 
By default everybody is allowed to execute at but those listed in /etc/cron.deny. If there is an allow file, only those people can execute at
Check if service is running
# systemctl status atd
$ at 17.45 23 May 2018
List jobs with
$ atd
Remove jobs
$ atrm

11 - LOG FILES AND LOGROTATE
* Auditing user logins
Last login of every users
# lastlog
Reverse filter 
# lastlog | grep -v "Never"
Extra information
# last -n 10
Users still logged
# last | grep "still"
Reboots
# last reboot
We can look for a particular user
# last tux 
# last -n 10 tux
To execute last bad login
# lastb
* Auditing root access
We can find logs at /var/log (/var/log/secure)
ls secure*
When we look at secure files we cannot audit su context. (sudo: su:)
When the file rotates, it does it with the date extension
* Scripting with AWK
$0 is the whole match
# awk '/sudo:/ { print $0 }' /var/log/secure
# awk '/sudo:/ { print $5, $6, $14}' /var/log/secure
Let's put into a script
* Configuring rsyslogd
Main configuration is at /etc/rsyslog.conf but can be extended with .conf files at rsyslog.d folder
/var/log/messages is definetly a place to look at when things go wrong. 
Facility local1.info /var/log/service_log at /etc/rsyslog.conf
logger -p local1.warn "Script ended"
* Using logrotate
Configuration at /etc/logrotate.conf
Command executed once a day /etc/cron.daily.
Manually rotate logs 
# logrotate /etc/logrotate.conf
* Managing logs with journalctl
CentOS is system D based. We can access all logs as
# journalctl 
Last 10/15 entries
# journalctl -n
# journalctl -n 15
From real  time
# journalctl -f
From the boot
# journalctl -b
For daemon logs
# journalctl -u sshd.service
# journalctl --since "2021-12-13 12:00:00"
# journalctl --since "10 minutes ago"
List boots
# journalctl --list-boots

12 - INTRODUCING SELINUX
* List and identify SELinux files and processes
Mandatory access control system. 
* SELinux status and contexts
Change the mode to permissive 
See the shadow context
# ls -Z /etc/shadow
----------. root root system_u:object_r:shadow_t:s0    /etc/shadow
Current mode
# getenforce
Check the SE status
# sestatus
Check configuration
# cat /etc/selinux/config
Be careful by enforcing and allowing. FS has to be changed as well. 
To change the current mode (Permissive)
# setenforce 0
getenforce will be Permissive
To enforce again
# setenforce 1
For user id 
# id -Z
For processes as well
# ps -Z
# ps -Zp $(pgrep sshd)
* Audit logs and SELinux commands
Check logs
# tail /var/log/audit/audit.log
To inspect logs for module avc (access vector control)
# ausearch -m avc
Change context for an object
# chcon -t unlabeled_t /etc/shadow
To restore the file
# restorecon /etc/shadow
To filter for recent events
# ausearch -m avc -ts recent
To see the SE context
# semanage fcontext -l
# semanage fcontext -l | grep /etc/shadow
For the targeted SE policies
cd /etc/selinux/targeted/contexts/files/
* Boolean and ports
For modifying policies by using booleans
Get installed policies
# getsebool -a | wc -l
More detail on it
# semanage boolean -l
# getsebool httpd_read_user_content
To change the policy 
# setsebool httpd_read_user_content on
And to change permanent
# setsebool -P httpd_read_user_content on
See port policies
# semanage port -l
# semanage port -l | grep ssh
To add a new port
# semanage port -a -t ssh_port_t -p tcp 2222

13 - MANAGING SOFTWARE ON CentOS7
* Use the rpm command
# yum install httpd
Into the yum cache /var/cache/yum/x86_64/7/base/packages/
Native RPM package. List all software
# rpm -qa 
To install a package
# rpm -i <name.rpm>
Remove the package
# rpm -e nmap
List content of an installed package
# rpm -ql nmap
The package wasn't installed
# rpm -qpl <name.rpm>
Verify the installed package
# rpm -V nmap
* Working with yum
Install bash completion with yum
# yum install bash-completion
To auto accept 
# yum install -y bash-completion
Get info about a package
# yum info bash-completion
Installed packages
# yum list installed
Remove software
# yum remove bash-completion
* Configuring software repositories
Repo files at /etc/yum.repos.d/
List repositories
# yum repolist
List all repositories (even the disabled ones)
# yum repolist all
* Working with the yum cache
Cache at /var/cache/yum
To clean the cache
# yum clean all
# yum makecache
* Updating the kernel
When updating the kernel yum will install new kernel and does not update the original. It updates grub to make the new one the default pick. This make it safe s we can always choose to boot to the original kernel. If we do not want to install new kernels or upgrae the existing we can add exclude=kernel* to /etc/yum.conf
* Using RPMS
Enable repos in CentOS-Source.repo
# yum install -y yum-utils ncurses-devel
# yum groupinstall "Development Tools"
# yumdownloader --source zsh
# rpm -i zsh..src.rpm 
# cd ~/rmpbuild/SOURCES
# tar -xjf zsh..tar.bz2
# cd zsh...; ./configure
# make; make install
* Managing Services
Install, enable and start services
# systemctl enable httpd.service
# systemctl start httpd.service
--> for both enable and start
# systemctl enable http.service --now
# systemctl status httpd.service
Remove services
# systemctl remove httpd.service

14 - CONFIGURATION MANAGEMENT TOOLS
* Install puppet (ruby based product)
# yum -install -y puppet
# puppet --version
* Extracting Data with facter
We can use facter to pass configuration detail to puppet
# facter | grep hostname
# facter | grep domain
* Using and applying manifests
Created at /etc/puppet/manifests/site.pp. Apply rules for files, packages, services and users. 
# puppet apply site.pp
