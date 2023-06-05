## WEB SOLUTION WITH WORDPRESS

Use RedHat OS for this project

### STEP 1: Prepare a Web Server
1. Launch an EC2 instance to serve as the web server. Create 3 volumes in the same zone as the Web Server, each of 10 gig
1. Attach all three volumes one by one to the Web Server.
1. Open the Linux terminal to begin configuration. Use the ```lsblk``` command to inspect what block devices are attached to the server. Notice the names of your newly created devices. All devices in Linux reside in /dev/directory. Inspect it with ```ls /dev/``` and make sure you see all 3 newly created block devices there – their names will be xvdf, xvdh, xvdg.
1. Use ```df -h``` command to see all mounts and free space on your server.
1. Use 'gdisk' utility to create a single partition on each of the 3 disks.
```sudo gdisk /dev/xvdf```. (*Repeat this for the other disks - xvdh and xvdg*).
1. Install 'lvm2' package using ```sudo yum install lvm2```. Run ```sudo lvmdiskscan``` command to check for available partitions.
*Note: In Ubuntu, 'apt' command is used to install packages. However, in RedHat/CentOS, a different package manager is used, so we will use 'yum' command instead.
1. Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1 
```
1. Verify that your Physical volume has been created successfully by running ```sudo pvs```
1. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG 'webdata-vg'.
```sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1```
1. Use lvcreate utility to create 2 logical volumes. apps-lv will use half of the PV size and logs-lv Use the remaining space of the PV size. (apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs).
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
1. Verify that your Logical Volume has been created successfully by running ```sudo lvs```
1. Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
1. Use mkfs.ext4 to format the logical volumes with ext4 filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
1. Create /var/www/html directory to store website files
```sudo mkdir -p /var/www/html```
1. Create /home/recovery/logs to store backup of log data
```sudo mkdir -p /home/recovery/logs```
1. Mount /var/www/html on apps-lv logical volume
```sudo mount /dev/webdata-vg/apps-lv /var/www/html/```
1. Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system).
```sudo rsync -av /var/log/. /home/recovery
Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)
sudo mount /dev/webdata-vg/logs-lv /var/log
Restore log files back into /var/log directory
sudo rsync -av /home/recovery/logs/. /var/log
Update /etc/fstab file so that the mount configuration will persist after restart of the server.











