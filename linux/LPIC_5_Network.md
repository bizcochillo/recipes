# 02 - INTRO
IP vs IFCONFIG

`$ ifconfig`

`$ ifconfig lo`

man pages show that the command is obsolete

`$ man ifconfig`

Use instead ip command

`$ ip addr`

`$ ip r`

`$ ip n`

It's a single command to show a lot of topics related to networking

We can create additional kernel network namespaces (e. g. namespaces for production and development machines). They run 

`$ sudo ip netns add development`

`$ ip netns`

# 03 - CONFIGURING HOSTNAMES

## Viewing hostnames

`$ echo $PS1`

`$ hostname`

`$ uname -n`

Show information

`$ hostnamectl`
## Configuring hostnames
To change the hostname, write

`# hostname centos7`

But it's not persistent (transient hostname). We have to change the name at /etc/hostname filename. Now things are different and we can use hostnamectl

`# hostnamectl set-hostname centos72.example.com`

`# hostname` => `centos72`

`# cat /etc/hostname` => `centos72`

To set the pretty name 

`# hostnamectl set-hostname "centos'72.example.com"`

To check with hostnamectl. We can see the pretty info at the `/etc/machine-info` file. The idea is to have this pretty name is for tables, mobiles, etc

`# cat /etc/machine-info`

## The local hosts files
We can use alias in the hosts file in the form of IP FQDN alias
## DNS name resolution
Service discovery service AVAHI (other systems use Bonjour for instance)

`# yum info avahi`

to see the hosts

`# getent hosts`

Check databases to see how to resolve dns queries

`# grep hosts /etc/nsswitch.conf`

To see for the DNS network server

`# cat /etc/resolv.conf`

Added by the network manager. Search looks for server names. Default domain to be looking at
To use the dig utility we have to install the bind-utils package 

`# yum install -y bind-utils`

`# dig`

To check DNS answers (by the default DNS server)

`# dig www.pluralsight.com`

To query a DNS server other than the default one. 

`# dig www.pluralsight.com @8.8.8.8`

A reduced version of the DNS responses

`# dig +short www.pluralsight.com @8.8.8.8`

To see the records

`# dig +short www.pluralsight.com @8.8.8.8 MX`

# 04 - CONFIGURING NTP

## Using date and hwclock

Differences between trasient time and hardware time. Hardware clock, system time or hwclock

System time 

`# date`

Hardware time

`# hwclock --systohc`

Synchronize in both directions

`# hwclock --hctosys`

## Using timedatectl

All systemd based system has timedatectl command. To show basic time information

`# timedatectl`

Failed to set time: Automatic time synchronization is enabled. To disable

`# timedatectl set-ntp false`

Afterwards we can change the time 

`# timedatectl set-time "2016-04-06 13:30:00"`

To disable ntp

`# timedatectl set-ntp false`

## Time synchronization with chronyd

Install chronyd, a newer datetime configuration tool for sync time

`# yum install -y chrony`

`# vim /etc/chrony.conf`

To see chrony configuration file

`# vim /etc/chrony.conf`

Enable and start the service

`# systemctl enable chronyd`

`# systemctl start chronyd`

`# systemctl status chronyd`

`# chronyc tracking`

## Time synchronization with ntpd

Traditional network time protocol. It uses the UDP port 123

ntp configuration

`# vim /etc/ntp.conf`

To see the time servers

`# ntpq -p`

# 05 - NETWORK SERVICES

## Network Services

IP addresses: v4 (32 bit). v6 (128 bit quad hex)

## Configuring IP addresses

Show IP address 

`$ ip addr show`

Reduced way

`$ ip a`

To show ip v4

`$ ip -4 a`

To show only one interface 

`$ ip -4 a s enp0s8`

To show ip v6 address

`$ ip -6 a`

To add a new IP address

`# ip addr add 172.17.67.3/16 dev enp0s8`

This information is transient and as soon as we restart, we loose the configuration

## Working with the NetworkManager

To display the status of the Services

`# systemctl status NetworkManager`

To use network manager we use the nmcli

To see the connection

`# nmcli connection show`

To see a specific interface

`# nmcli connection show enp0s8`

Pretty formatting

`# nmcli -p connection show enp0s3`

To add a new connection (home) to a physical interface 

`# nmcli connection add con-name home ifname enp0s3 type ethernet ip4 192.168.0.98 gw4 192.168.0.1`

Once home create we can put down the connection

`# nmcli con down enp0s3`

And put up the new home

`# nmcli con up home`

And then, back again.

## Working with Network Configuration Files

Standard network Services. To check the Services

`# systemctl status network`

the configuration comes from the /etc/sysconfig/network-scripts/ directory

To not be managed by the network manager NM_CONTROLLED=no

We have to configure the /etc/sysconfig/network-scripts/ifcfg-<INTERFACE-NAME>

  >> IPADDR=""
  >> NETMASK=""
  >> DNS1=""
  >> DNS2=""
  >> NM_CONTROLLED="no"

`# ifdown enp0s8`

`# ifup enp0s3`

# 06 - ROUTING WITH CENTOS7

## Display Route Tables (manage everything through a single command)

`# ip route show`

Print the route table

`# route`

`# netstat -r`

`# netstat -nr`

## Adding Routes

`# ip route add default via 192.168.56.102`

By reboot we lost the changes

We make the changes persistent by adding the gateway option `"GATEWAY"="192.168.56.102"` at `/etc/sysconfig/network-scripts/ifcfg-enp0s8`

## Enabling Routes / Configure a Linux system as a router.

In server2 we need to change the ip forwarding option `/etc/sysctl.conf`

`net.ipv4.ip_forward=1`

## Enabling NAT

We stop the firewalld

`# systemctl stop firewalld`

To see the iptables

`# iptables -L`

To see the NAT ROUTING

`# iptables -t nat -L`

Add a route to find the way to the interface with outbound access 

`# iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE`

Now, from server1 we have access to internet. 

# 07 - FIREWALLING WITH firewalld

## Firewall zones
- Install firewalld with `yum install -y firewalld`
To check firewall status

`# firewall-cmd --state`

`# systemctl start firewalld`

To check the default zones

`# firewall-cmd --get-default-zone`

To get the active zones

`# firewall-cmd --get-active-zones`

`# firewall-cmd --get-zones`

To put the interfaces in the correct zones

`# firewall-cmd --permanent --zone=public --remove-interface=enp0s3`

`# firewall-cmd --permanent --zone=public --remove-interface=enp0s8`

`# firewall-cmd --permanent --zone=external --add-interface=enp0s3`

`# firewall-cmd --permanent --zone=internal --add-interface=enp0s8`

`# firewall-cmd --get-active-zones`

Set the default zone

`# firewall-cmd --set-default-zone=external`

`# systemctl restart firewalld`

And default zone returns external

## Working with Services

Firewall in relying on iptables. To see firewall information

`# firewall-cmd --list-all`

`# firewall-cmd --list-all --zone=internal`

To remove ipp-client

`# firewall-cmd --permanent --remove-service=ipp-client --zone=internal`

To add nfs service

`# firewall-cmd --permanent --add-service=nfs --zone=internal`

`# firewall-cmd --reload`

## Custom Services

To only enable a few Services

`# firewall-cmd --remove-service={dhcpv6-client,mdns,samba-client} --zone=internal --permanent`

`# firewall-cmd --reload`

Add Services

`# firewall-cmd --add-service={high-availability,http} --zone=internal --permanent`

`# firewall-cmd --reload`

`# firewall-cmd --list-services --zone=internal`

To check files with firewall rules

`# ls /usr/lib/firewalld/services`

`# firewall-cmd --permanent --new-service="puppet"`

`# restorecon puppet.xml`

`# chmod 640 puppet.xml`

`# vi puppet.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
    <short>Puppet</short>
    <port protocol="tcp" port="443" />
    <port protocol="tcp" port="8140" />
</service>
```
`# firewall-cmd --permanent --add-service=puppet --zone=internal`

`# firewall-cmd --reload`

`# firewall-cmd --list-services --zone=internal`

## NAT and Masquerade with firewalld

To remove masquerade (Server2)

`# firewall-cmd --permanent --remove-masquerade`

`# firewall-cmd --reload`

To add masquerade (server2)

`# firewall-cmd --permanent --add-masquerade`

`# firewall-cmd --reload`

# 08 - USING iptables

## iptables toolset

Used by many firewall systems

`iptables -L` will show all the rules configured for networking.

*Example*: save firewall off rules (fwoff, no rules). Create few rules and save firewall on (fwon, three rules)

- Save current firewall status

`iptables-save > fwoff`
  
- Add an input rule from the localhost interface (lo) for jumping to accept everything (Accept everything coming to the localhost. 
  
`iptables -A INPUT -i lo -j ACCEPT`
  
- Add an input rule for a module (connection track) for a connection track states established and related. The traffic will be allow to come back.
  
`iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT`
 
- Rule to allow traffic to the port 22
  
`iptables -A INPUT -p tcp --dport 22 -j ACCEPT`

- Save rules on in a file. 

`iptables-save > fwon`

- To restore either of the rules we use `iptables-restore` (toggling firewall on and off)
 
`iptables-restore < fwoff` 

`iptables-restore < fwon`

To show the iptables rule information along with the interfaces 
  
`iptables -nvL`
  
It's possible to add a drop rule to the firewall to avoid accepting everything: `-A INPUT -j DROP` is to add in the fwon file. 
## firewall design
  
  To flush the iptables we use. It will drop all rules from the iptables.
  
  `iptables -F`
  
  To do not forward any traffic, add `-A FORWARD -j DROP` to the fwon file
  
## install iptables service
  
  To install the iptables services and configure the service with the tool
  
  `yum install -y iptables-services`
  
  Once the service is installed, we can find the configuration at `/etc/sysconfig/iptables`. There we can see the rules. We got another configuration file at `/etc/sysconfig/iptables-config` for configuring the behavior of the service. Pay attention to the setting `IPTABLES_SAVE_ON_RESTART="yes"`.
  
  We can replace the firewall for the iptables service: 
  
  ```console
  systemctl stop firewalld.service
  systemctl enable iptables.service
  systemctl start iptables.service
  # We see now that the rules have been imported to iptables 
  iptables -L
  ```

  
# 09 - METHODS TO TUNNEL TRAFFIC
# 10 - MONITORING THE NETWORK 
