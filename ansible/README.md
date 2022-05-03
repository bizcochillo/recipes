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
