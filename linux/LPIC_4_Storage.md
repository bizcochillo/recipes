# 01 - INTRODUCTION

## Listing Block Storage

To list disk blocks issue `lsblk`. `sr0` is the CD rom. `lvm` is a logical volume. swap is virtual memory. MAJ and MIN numbers are really important. Maj number indicated the version number of the driver that the kernel is using to access the disk. 

The driver allows up to 15 partitions per disk. No matter if we're using MBR partitions or GUI, that allows up to 128, but the limitation comes from the driver. 

# 02 - PARTITIONING DISKS

## Partitioning Linux disks

```console
DISK -> 
	MBR Partition Table 2TB (maximum partition table) ->
		Primary Max partition 4. 
		Logical Partition (limited by the driver the operating system support. SCSI driver limited to 15 partitions)
	GUID Partition Table 8ZB ->
		Partition Max 128
```

File system on top partitions: ext2, ext3, ext4, ReiserFS, XFS, Btrfs

## Using fdisk to create partitions

To list the partitions that are in a file disk 

```console
fdisk -l /dev/sdb
```

To create a partition in interactive mode

```console
fdisk /dev/sdb
```

Remove the partition by overwritting the partition root

```console
dd if=/dev/zero of=/dev/sdb count=1 bs=512
```

## Using gdisk to create partitions

To create partitions with gdisk

```console
gdisk /dev/sdb
```

There is a backup at the end of the disk (First 512 is the MBR, seconds 512 for the GPT headers, GPT partition for the 16K and the last 16K of the disk are the partition backup. 

## Using GNU parted to create partitions

```console
parted /dev/sdb printmk
```

To create a label

```console
(parted) mklabel msdos
```

To create a primary partition (200 MB)

```console
(parted) mkpart primary 1 200
```

To create an extended partition to whole disk

```console
(parted) mkpart extended 201 -1
```

To create a logical partition 

```console
(parted) mkpart logical 202 300
```

Be aware of the numbering, as MBR only allows up to 15 partitions. 

## Scripting partition creation

parted can script creation of partitions. parted -s is the silent mode. -- is required if we used -1m at the end. 
Backing up the MBR (dos label disk)

```console
dd if=/dev/sda count=1 bs=512 of=/root/sdb.mbr
```

## Codes for partitions

MSDOS Label: 83 
Linux, 82 
swap, 8e LV, fd RAID
GPT 8300 Linux, 8200 swap, 
Copy held at the end of the disk. 

# 03 - CREATING LINUX FILE SYSTEMS

## General purpose file systems with ext4

`mkfs.<tab>` show the options

To create a file system and add a label DATA

```console
mkfs.ext4 -L DATA /dev/sdb6
```

To modify file system settings

```console
tune2fs -L MYDATA -c 0 -i 0 /dev/sdb6
```

To show metadata of a volume

```console
dumpe2fs /dev/sdb6
```

## Enterprise class file systems with xfs

To create (originally developed by Silicon Graphics)

```console
mkfs.xfs -b size=1k -l size=10m /dev/sdb7
```

To go into expert mode into xfs

```console
xfs_db -x /dev/sdb7
```

To get help

```console
xfs_db> help
```

To get UUID
```console
xfs_db> uuid
```

Get label

```console
xfs_db> label
```

Set label to DATA2

```console
xfs_db> label DATA2
```

## Using the mount command and ext4 file systems

Mount ext4 to directory

```console
mount /dev/sdb6 /mnt
```

Unmount the fs
```console
umount /mnt
```

Create directories and mount file systems

```console
mkdir -p /data/{mydata,data2}
mount /dev/sdb6 /data/mydata
mount | grep mydata
```console

We can add options to the mount command 

```console
mount -o remount,noexec /dev/sdb6 /data/mydata
```

mount shows the content of the process mount

```console
cat /proc/mounts
```

To get the label/id of a volume

```console
blkid /dev/sdb6
```

To go to the /etc/fstab file to mount (to persists the mounts) 

```console
vi /etc/fstab

(in the /etc/fstab) 
UUID="8d82e24b-0799-4421-8e46-1c0e6adda87c" /data/mydata ext4 noexec 0 2
<CTRL>+U to delete the line
```

Afterwards we can mount all
```console
mount -a 
```

## Using the mount command and xfs file systems

The same for xfs system

```console
UUID="0d6cf89b-2acf-41bd-8979-1a2ac2b068c0" /data/data2 xfs defaults 0 0
# mount -a
```

Everything should be fine

```console
mount | grep data2
```

Once mounted, we can see metadata

```console
xfs_info /dev/sdb7
```

## Mount options

Look at man pages and go for `/INDEPENDENT`. Independent features, no access time, enabling user quota and group quota only for informative purposes. 
```console
In /etc/fstab:
  UUID="8d82e24b-0799-4421-8e46-1c0e6adda87c" /data/mydata ext4 noatime,noexec 0 2
  UUID="0d6cf89b-2acf-41bd-8979-1a2ac2b068c0" /data/data2 xfs noexec,noatime,uquota,gqnoenforce 0 0
```console

# 04 - MANAGING SWAP AND RAID SERVICES
# Creating swap space

Create a swap partition

```console
mkswap /dev/sdb5
```

We can see the swap Summary

```console
swapon -s
```

The priority can control how the swap space is used. To set the partition to be used as swap

```console
swapon /dev/sdb5
```

To disable swap on a swap partition

```console
swapoff /dev/sdb5
```

To see how many free memory swap has

```console
free -m
```

We also can use swap files. Here we have been using swap partitions.

## Setting the swap priority and mounting at boot

To disable swap at all

```console
swapoff -a
```

To configure swap look at the `/etc/fstab` file. Two example lines:
```console
>> /dev/mapper/centos-swap swap                    swap    sw,pri=1 0 0
>> UUID="69d16c94-5bb9-4918-9347-4afaa4f6f3c1" swap swap sw,pri=5 0 0
```

## Configuring software RAID

RAID: Redundant Array of Inexpensive Disks. Usually used to create fault tolerant disk arrays. Traditional Linux Software RAID creates dev mapper virtual device file. RAID is integrated into Btrfs

Linear: Partitions/disk not the same size, volume is expanded across all disk in array. Spare disk is not supported

RAID0: Similar to above but the disk or partitions are similar size. 

RAID1: Mirror data between devices of similar size

RAID4/5/6: Three or more devices (4 with RAID6), data is striped with parity

To check if we support RAID

```console
cat /proc/mdstat
```

Create a raid based on the raid enabled partitions with the multi disk administrator tool (level mirror)

```console
mdadm --create --verbose /dev/md0 --level=mirror --raid-devices=2 /dev/sdb13 /dev/sdb14
```

To see if the Linux Kernel modules have been Loaded

```console
lsmod | grep raid
```

Create file system on the new created partition

```console
mkfs.xfs /dev/md0
```

Check the status of the RAID 

```console
mdadm --detail --scan
mdadm --detail --scan >> /etc/mdadm.conf
mdadm --stop /dev/md0
mdadm --assemble --scan 
```

# 05 - EXTENDING PERMISSIONS WITH ACLS

## ACL support within the Kernel and file systems

To see levels of ACLs support

```console
grep ACL /boot/config-$(uname -r)
```

Option y indicates that it's compiled into the kernel and m it's loaded in a module by the kernel. 
To show the file system characteristics and acl is supported

```console
sudo tune2fs -l /dev/sdb6 | grep -i default
```

Depending the file system it may be necessary to load the kernel module
```console
drwxrwxr-x.  3 tux  tux         18 Nov 16 13:17 usr
```

## Listing file system ACLs
The . (dot) after the list indicates support for the ACLs but it's currently not set
To see the acls of a file

```console
getfacl df.sh
```

Others does not have access to the directory test-acl

## Setting default ACLs

```console
setfacl -m d:o:--- test-acl/
```

To add ACLs for other user

```console
setfacl -dm u:bob:rw test-acl/
```

## Adding ACL entries

```console 
mkdir /work
```

```console
chmod o= /work/
```

To add ACLs for individual files, for instance

```console
setfacl -m d:o:--- /work
echo file1 > /work/file1
echo file2 > /work/file2
setfacl -m u:tux:rw /work/file1
su - tux
cd /work/

==> It's ok

echo hello >> file1
cat file1

==> Does not work (permission denied)
cat file2
```


## Removing ACLs

To remove individual entries

```console
setfacl -x u:tux file1
```

To remove all entries

```console
setfacl -b file1
```

## Diagnosing and resolving security issues

See password aging attributes for the user

```console
chage -l tux
```
To change the context in SELinux we cannot see the file

```console
ls -Z /etc/shadow
```

Install web server httpd

```console
yum install httpd
mkdir /web
chgrp apache /web
chmod 2750 /web/
setfacl -m d:o:--- /web
echo "My Web" > /web/index.html
ls -l /web/index.html
vim /etc/httpd/conf/
apachectl configtest
systemctl restart httpd
```

To look for events

```console
ausearch -m AVC -ts recent
```

We get the right web page but the wrong context

```console
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
```

Restore content in directory and files within

```console
restorecon /web
restorecon /web/*
```

Check context is correct

```console
ls -Zd /web/
ls -Z /web/
```

# 06 - MANAGING LOGICAL VOLUMES

## Creating LVM in CentOS

LVM. (PV, VG and LV). We can create a lot of necessary Logical Volumes

To see the physical volumes

```console
pvscan
```

To se the volume groups

```console
vgscan
```

To see the logical volumes on top of it

```console
lvscan
```

At fdisk look at the 8e partitions Linux LVM. Create partitions

```console
pvcreate /dev/sdb1[1-2-3]
```

Create a volume group

```console
vgcreate vg1 /dev/sdb10 /dev/sdb11
```

To see info

```console
vgscan 
```

And even more information and create partition

```console
vgs
lvcreate -n lv1 -L 184m vg1
```

And create a file system

```console
mkfs.xfs /dev/vg1/lv1
```

Mount the file system

```console
mkdir /lvm
vi /etc/fstab
 >> /dev/vg1/lv1 /lvm xfs defaults 0 0
mount -a
```
We copy some documents to the new mounted directory

```console
find /usr/share/doc -name '*.pdf' -exec cp {} /lvm/ \;
```

We see the usage

```console
df -h
```

## Resizing logical volumes on the fly

To extend a volume group. We start seeing what is going on:

```console
pvscan
vgextend vg1 /dev/sdb12
```

Extend the logical volume
```console
lvextend -L +50m /dev/vg1/lv1
```

We have to resize the file system as well

```console
xfs_growfs /lvm
```

## LVM snapshots

Create a backup volume
```console
lvcreate -L 30m -s -n backup /dev/vg1/lv1
mount /dev/vg1/backup /mnt -o nouuid,ro
```

We can see the files in mnt

```console
ls /mnt
```

We can put the backup 
```console
tar -cf /root/backup.tar /mnt
umount /mnt
lvremove /dev/vg1/backup
```

## Migrating PVs to new storage

Create a new storage. References in fstab will remain

```console
pvcreate /dev/sdc5
```
Extend the original volume

```console
# vgextend vg1 /dev/sdc5
# pvmove /dev/sdb10 /dev/sdc5
```

Reduce and wipe the volumes

```console
vgreduce vg1 /dev/sdb10
pvremove /dev/sdb10
```
Resize the logical volume

```console
vgs
```

# 07 - CONFIGURING AN ISCSI BLOCK STORAGE SERVER

# 08 - IMPLEMENTING HA CLUSTERS

# 09 - IMPLEMENTING AGGREGATED STORAGE WITH GlusterFS

# 10 - ENCRYPTED VOLUMES

# 11 - USING THE AUTO/MOUNTER

# 12 - IMPLEMENTING USER AND GROUP QUOTAS
