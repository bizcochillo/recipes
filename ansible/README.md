Reference: https://app.pluralsight.com/library/courses/hands-on-ansible/table-of-contents

# Requirements

Python 2.6+ 
Linux/Unix/Mac(Windows not supported). 
For control server Windows is not supported
For remote server windows is allowed with Powershell 2

Inventory (text files which defines servers, variables groups and roles)
Module (A programmed unit of work to be done)
PlayBook (Glue which fits all together)
A Play a single or set of tasks using modules, executed on a defined set of hosts. A PlayBook is a set of plays built in specific order sequence to produce an expected outcome or outcomes across many different sets of hosts. 
Ansible Config. Global configuration settings.
Three kinds of variables: 
- Host Variables: Use variables defined in inventory per host or group
- Facts: Use data gathered from the remote managed host
- Dynamic variables: Use data gathered by tasks or created at runtime

Execution type: remote (executed on remote server) and local (Execute at control server)

# Environment
To list all running VM with Virtual Box

```console
vboxmanage list runningvms
```

Install Ansible on acs (Ubuntu)

```console
sudo apt-get install -y ansible
```

Install Ansible on `web` and `db`. For CentOS systems, the repository package has changed, follow the instructions on (https://proyectoa.com/solucion-al-error-failed-to-download-metadata-for-repo-appstream-en-centos-8/):
  
  - Change the repository references
    
    ```console
    sudo sed -i -e "s|mirrorlist=|#mirrorlist=|g" /etc/yum.repos.d/CentOS-*
    sudo sed -i -e "s|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g" /etc/yum.repos.d/CentOS-*
    ```
    
  - Clean previous cache
    
    ```console
    sudo dnf clean all
    ```
    
  - Update repository
   
    ```console
    sudo dnf swap centos-linux-repos centos-stream-repos
    ```   
On <u>web</u> we can now use yum. First we install epel-release package and the ansible package

```console
sudo yum install -y epel-release
sudo yum install -y ansible
```
On <u>db</u> we can install ansible by compiling the python code. First, we make sure that we have gcc and install the python-setuptools (for python2)

```console
sudo yum install -y gcc
sudo yum install -y python2
sudo yum install -y python2-devel
sudo pip2 install ansible
```

Exercise 1 is a very simple setup where an inventory file composed by only the IP's of the web and db servers is requested for performing a ping operation with the ping module (`-m ping`), but before, we need to enable password authentication at SSH level (file `/etc/ssh/sshd_config` uncomment the line with `PasswordAuthentication yes` and restart the ssh service with `systemctl restart sshd`. So for setup the exercise: 

```console
mkdir exercise1 && cd exercise1
cat <<EOF > inventory
192.168.33.20
192.168.33.30
EOF
```

For querying the web server:

```console
ansible 192.168.33.20 -i inventory -u vagrant -m ping -k
```

For querying the db server: 

```console
ansible all -i inventory -u vagrant -m ping -k
```

For running ansible with level 3 debugging: 

```console
ansible 192.168.33.20 -i inventory -u vagrant -m ping -k -vvv
```

Other examples for executing a command with the command module

```console
ansible all -i inventory -u vagrant -m command "/bin/reboot"
``` 

Is equivalent to 

```console
ansible all -i inventory -u vagrant "/bin/reboot"
``` 

Basic ansible commands `ansible <system> -i <inventoryFile> -m <module> -u <username> -k <password prompt> -v (-vv debug level2 / -vvv debug level3)`

# Inventory and Configuration

Inventory files can contain Behavioral parameters, Groups, Groups of Groups, Assign variables, can be scaled out by using multiple files and can be static or dynamic. We define the items and the variables attached to the host. In the example below we set the host, and the SSH access in plain text. (Filename: `inventory`)

```dosini
web1 ansible_ssh_host=192.168.33.20 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
db1 ansible_ssh_host=192.168.33.30 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
[webservers]
web1
db1
```

We can execute modules (simple ping) for the individual server as follow:

```console
ansible web1 -i inventory -m ping
```

And for the group `webservers`:

```console
ansible webservers -i inventory -m ping
```

With the keyword `:children` we can use groups of groups. We move `db1` to the `dbservers` group and create a `datacenter` group of groups containing `webservers` and `dbservers` as follow:

```dosini
web1 ansible_ssh_host=192.168.33.20 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
db1 ansible_ssh_host=192.168.33.30 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant

[webservers]
web1

[dbservers]
db1

[datacenter:children]
webservers
dbservers
```

Additionally, we can define variables for using in scope of a group. For this, we use the keyword `:vars`:

```dosini
web1 ansible_ssh_host=192.168.33.20
db1 ansible_ssh_host=192.168.33.30

[webservers]
web1

[dbservers]
db1

[datacenter:children]
webservers
dbservers

[datacenter:vars]
ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
```

When we have a multi file inventory setup, the order of operations (precedence) is:

1. (Group_Vars) All
2. (Group_Vars) GroupName
3. (Host_Vars) HostName

Typical structure is:

```console
.
├── group_vars
│   ├── all
│   └── webservers
├── host_vars
│   └── web1
└── inventory_prod
```

Variable files are written in YAML:

```yaml
---
# file: group_vars/dc1-west
ntp: ntp-west.company.com
syslog: logger-west.company.com
```

For instance, with the command 

```console
ansible webservers -i inventory_prod  -m user -a "name={{username}} password=12345" --sudo
```

Where:
- `-m user`: specifies the module user for creating users 
- `-a "name={{username}} password=12345`: user to be retrieved from variables (variable `username`) and password given in clear text. 
- `--sudo`: to give permissions to the `useradd` command executed by the module. 

File `group_vars/all`

```yaml
---

# This is our user for all groups
username: all_username
```

File `group_vars/webservers`

```yaml
---

# This is for webservers group
username: group_user
```

File `host_vars/web1`

```yaml
---

# Only for web1 server
username: web1_user
```

We can have as output, while aggregating sequentially the files `group_vars/all`, `group_vars/webservers` and `host_vars/web1`:

```console
web1 | success >> {
    "changed": true,
    "comment": "",
    "createhome": true,
    "group": 1001,
    "home": "/home/all_username",
    "name": "all_username",
    "password": "NOT_LOGGING_PASSWORD",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1001
}
```

```console
web1 | success >> {
    "changed": true,
    "comment": "",
    "createhome": true,
    "group": 1002,
    "home": "/home/group_user",
    "name": "group_user",
    "password": "NOT_LOGGING_PASSWORD",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1002
}
```

```console
web1 | success >> {
    "changed": true,
    "comment": "",
    "createhome": true,
    "group": 1003,
    "home": "/home/web1_user",
    "name": "web1_user",
    "password": "NOT_LOGGING_PASSWORD",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1003
}
```

Ansible Configuration component defines how ansible works (e. g. how many parallel operations does ansible has). The order of precedence for configuration values are: 

1. `$ANSIBLE_CONFIG` variable. 
2. ./ansible.cfg file
3. ~/.ansible.cfg file
4. /etc/ansible/ansible.cfg file (Only exists if installed ansible with package manager like pip)

Configuration are not merged, the first one wins. It's possible to override certains parameter by environment variables with the prefix `ANSIBLE_<configsetting>` (e.g. `#export ANSIBLE_FORKS=10`. 

Some common settings:
- `[defaults] forks`: Total number of parallel operations ansible executes. (default to 5, recommendation 20 depending on perf)
- `[defaults] host_key_checking`: Checks if the system is valid and authorized. Recommended for production environments (default true, dev environments recommend to false). 
- `[defaults] log_path`: Write information on ansible executions (default to null). 

Docs on https://docs.ansible.com/

# Modules

Ansible modules are the building block that makes automation. There are three types of modules: core, extra and deprecated. Ansible has installed locally documentation for those modules. To browse the modules issue `ansible-doc -l`, to get specific information about a module `ansible-doc <name>` and for playbook examples `ansible-doc -s <name>`. Some common modules are: 
- Copy Module. Copies a file from local box to remote system. Has "backup" capability. Can do validation remotely. 
- Fetch Module. Pulls a file from remote host to local system. Can use md5 checksums to validate. 
- Apt Module. Manages installed applications on Debian based system. Can install, update or delete packages. Can update entire system.
- Yum Module. Same thing for Red Hat-based systems.
- Service Module. Can stop, start or restart services. Can enable services to start on boot

>*NOTE*: With `ansible-doc -l` we can navigate down and up with the arrows and page 

