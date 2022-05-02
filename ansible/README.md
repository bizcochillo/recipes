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
```va

Install Ansible

```console
sudo apt-get install -y ansible
```