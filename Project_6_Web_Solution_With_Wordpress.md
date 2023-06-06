## WEB SOLUTION WITH WORDPRESS

Use RedHat OS for this project

### STEP 1: Prepare a Web Server
1. Launch an EC2 instance to serve as the web server. Create 3 volumes in the same zone as the Web Server, each of 10 gig
2. Attach all three volumes one by one to the Web Server.
3. Open the Linux terminal to begin configuration. Use the ```lsblk``` command to inspect what block devices are attached to the server. Notice the names of your newly created devices. All devices in Linux reside in /dev/directory. Inspect it with ```ls /dev/``` and make sure you see all 3 newly created block devices there – their names will be xvdf, xvdh, xvdg.
4. Use ```df -h``` command to see all mounts and free space on your server.
5. Use 'gdisk' utility to create a single partition on each of the 3 disks.
```sudo gdisk /dev/xvdf```. (*Repeat this for the other disks - xvdh and xvdg*).
6. Use lsblk utility to view the newly configured partition on each of the 3 disks.
7. Install 'lvm2' package using ```sudo yum install lvm2```. Run ```sudo lvmdiskscan``` command to check for available partitions.
*Note: In Ubuntu, 'apt' command is used to install packages. However, in RedHat/CentOS, a different package manager is used, so we will use 'yum' command instead.
8. Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1 
```
9. Verify that your Physical volume has been created successfully by running ```sudo pvs```
10. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG 'webdata-vg'.
```sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1```
11. Use lvcreate utility to create 2 logical volumes. apps-lv will use half of the PV size and logs-lv Use the remaining space of the PV size. (apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs).
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
12. Verify that your Logical Volume has been created successfully by running ```sudo lvs```
13. Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
14. Use mkfs.ext4 to format the logical volumes with ext4 filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
15. Create /var/www/html directory to store website files
```sudo mkdir -p /var/www/html```
16. Create /home/recovery/logs to store backup of log data
```sudo mkdir -p /home/recovery/logs```
17. Mount /var/www/html on apps-lv logical volume
```sudo mount /dev/webdata-vg/apps-lv /var/www/html/```
18. Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system).
```sudo rsync -av /var/log/. /home/recovery```
19. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)
```sudo mount /dev/webdata-vg/logs-lv /var/log```
20. Restore log files back into /var/log directory
```sudo rsync -av /home/recovery/logs/. /var/log```
21. Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file. RUn the code below to get the UUID.
```sudo blkid```
```sudo vi /etc/fstab```
Update /etc/fstab in the format shown below using your own UUID and rememeber to remove the leading and ending quotes.
22. Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```
23. Verify your setup by running ```df -h```. The output must look like this:

### STEP 2: Prepare the Database Server
1. Launch a second RedHat EC2 instance that will have a role – ‘DB Server’.
2. Repeat the same steps as for the Web Server.
3. When you get to step 11, instead of apps-lv, create db-lv logical volume.
4. For step 15, create /db directory instead of /var/www/html/.
5. In step 17, mount db-lv logical volume to /db.
```sudo mount /dev/webdata-vg/db-lv /db```

*Tip: Incase you made a mistake and did not use db-lv instaed of apps-lv and mount to /var/www/html/ instead of /db, you can
use the lvrename command and re-mount to the correct directory*
```lvrename /dev/webdata-vg/apps-lv /dev/webdata-vg/db-lv










