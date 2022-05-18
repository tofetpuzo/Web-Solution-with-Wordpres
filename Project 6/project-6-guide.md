# Implementation of a three-tier architecture 
Generally most architectures are based on three tiers which are
a. Presentation layer: this is usually the user interface such as the client server or browser
b. Business layer: This is usually the backend program which implements the business logic side of things.
c. Data Access or Management layer: This is the layer for computer data storage and data access. Database Server or file system FTP.


## Creating a Web-Server.
The first step is to create an EC2 instance (Linux DISTR0: Centos8), after this I created 3 volumes of the EBS(elastic block store) and attached it to the EC2 instance => Web Server. 

![web](./images/block.png)

The configuration of the EBS was done through the AWS interface and validate in the console of the web server using the "lsblk" command to inspect what block devices were attached to the server.

![ebs](./images/ebs_blk.png)

The web server volumes were verified using the command `ls /dev/`
![elastic](./images/blks.png)

The next step, I took was to use the `df -h` command to see all mounts and free space on the web server. The output is shown below after running this command.
![elastic](./images/disk.png)


## Creating a Partition for a Web-Server.

The first step to create a partition for Linux distro(Centos 8) is to use this command 

`sudo fdisk /dev/xvdf`. 

This opens a partition interface for creating a partition
![elastic](./images/part_inf.png)

The next command is "p" this shows the information about that particular disk or volume.
![elastic2](./images/part.png)

To create a new partition , the command "n" was used and this was done for the other 2 volumes that was attached to this particular web server.
The next command is to put p which is the primary
![elastic3](./images/create_new_volume.png)

To effect the change the "w" command has to be used. 
![elastic4](./images/complete.png)

The last step is to use the `lsblk` command to see that the partition has been correctly specified. 
![elastic5](./images/partitioncomplete.png)


### Creating a Physical Partition for a Web-Server
The next step is to install lvm2 on the web-server using this command
`sudo yum install lvm2` 

`sudo pvs`

![elastic6](./images/pvc_created.png)


### Creating a Volume group for a Web-Server

After the physical volumes are created, these PV are then added to a volume group(VG). I named the VG : webdata-vg, by running this command.

`sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`
![elastic7](./images/vgs.png)


### Creating a Logical volumes for a Web-Server

The next step is to use lvcreate utility to create a two logical volumes such as apps-lv and logs-lv for the remaining space of the pv size.
`sudo lvcreate -n apps-lv -L 1.5G webdata-vg`


`sudo lvcreate -n logs-lv -L 1.5G webdata-vg`

To verify the entire setup
`sudo vgdisplay -v` #view complete setup -VG, PV and LV

`sudo lsblk`

![elastic7](./images/lv-complete.png)


The next step is to create a file system, the ext4 is used to format the logical volumes with ext4 filesystem.

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`


## Creating directory to store website files for a Web-Server.

The first step is to create a directory using this command.
`sudo mkdir -p /var/www/html`

The next step is to create a recovery directory to store backup of log data
`sudo mkdir -p /home/recovery/logs`

After creating, these directories the next step is to mount directory to apps-lv logical volume using this command.
`sudo mount /dev/webdata-vg/apps-lv /var/www/html`

Before doing, this above command we have to use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs 

`sudo rsync -av /var/log/. /home/recovery/logs/`

The step is to mount /var/log on logs-lv logical volume. 
`sudo mount /dev/webdata-vg/logs-lv /var/log`

The /etc/fstab file must be update so that the mount configuration will persist after restart of the server.

The next crucial step is to input all the blk-ids to the /etc/fstab in the format in the screenshot below, but first run this command to get all the ids. 
`sudo blkid`

`sudo vi /etc/fstab`

![elastic9](./images/fstab.png)

