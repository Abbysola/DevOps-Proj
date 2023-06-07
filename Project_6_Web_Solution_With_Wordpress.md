## WEB SOLUTION WITH WORDPRESS

Use RedHat OS for this project

### STEP 1: Prepare a Web Server
1. Launch an EC2 instance to serve as the web server. Create 3 volumes in the same zone as the Web Server, each of 10 gig.
![](https://github.com/Abbysola/DevOps-Proj/blob/8e0565f09ee635f70e6592051e01f006191f935e/Images/P6_Creating%20Volume.png)

2. Attach all three volumes one by one to the Web Server.
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Attach%20volume.png)

3. Open the Linux terminal to begin configuration. Use the ```lsblk``` command to inspect what block devices are attached to the server. Notice the names of your newly created devices. All devices in Linux reside in /dev/directory. Inspect it with ```ls /dev/``` and make sure you see all 3 newly created block devices there – their names will be xvdf, xvdh, xvdg.
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Checking%20Block%20Devices.png)

4. Use ```df -h``` command to see all mounts and free space on your server.

5. Use 'gdisk' utility to create a single partition on each of the 3 disks.

```sudo gdisk /dev/xvdf```
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Creating%20Single%20Partition.png)
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Disks.png)

(*Repeat this for the other disks - xvdh and xvdg*)

6. Use lsblk utility to view the newly configured partition on each of the 3 disks.
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_All%20partitions.png)

7. Install 'lvm2' package using ```sudo yum install lvm2```. Run ```sudo lvmdiskscan``` command to check for available partitions.
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Using%20lvmdiskscan%20to%20check%20available%20partitions.png)

*Note: In Ubuntu, 'apt' command is used to install packages. However, in RedHat/CentOS, a different package manager is used, so we will use 'yum' command instead.*

8. Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1 
```
9. Verify that your Physical volume has been created successfully by running 

```sudo pvs```
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Verifying%20physical%20volume%20created.png)

10. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG 'webdata-vg'.

```sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1```
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Verifying%20vg%20created.png)

11. Use lvcreate utility to create 2 logical volumes. apps-lv will use half of the PV size and logs-lv Use the remaining space of the PV size. (apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs).
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
12. Verify that your Logical Volume has been created successfully by running 

```sudo lvs```

![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Verifying%20lv%20created.png)

13. Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Verifying%20set-up.png)

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

![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_UUID%20check.png)

```sudo vi /etc/fstab```
Update /etc/fstab in the format shown below using your own UUID and rememeber to remove the leading and ending quotes.
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Editing%20UUID%20in%20%3Aetc%3Afstab.png)

22. Test the configuration and reload the daemon

```
sudo mount -a
sudo systemctl daemon-reload
```

23. Verify your setup by running ```df -h```. The output must look like this:
![](https://github.com/Abbysola/DevOps-Proj/blob/c9e581cef280339fcc0ffd63bc8d691241163b33/Images/P6_Verifying%20set-up.png)

### STEP 2: Prepare the Database Server
1. Launch a second RedHat EC2 instance that will have a role – ‘DB Server’.
2. Repeat the same steps as for the Web Server.
3. When you get to step 11, instead of apps-lv, create db-lv logical volume.
4. For step 15, create /db directory instead of /var/www/html/.
5. In step 17, mount db-lv logical volume to /db.

```sudo mount /dev/webdata-vg/db-lv /db```

*Tip: Incase you make a mistake and did not use db-lv instaed of apps-lv and mount to /var/www/html/ instead of /db, you can
use the lvrename command and re-mount to the correct directory*
1. Check the current logical volumes

```lsblk```

2. Unmount the wrong logical volume

```umount /mnt/apps-lv```

3. Rename the logical volume

```lvrename /dev/webdata-vg/apps-lv /dev/webdata-vg/db-lv```

4. Enable the logical volume

```lvchange -a y /dev/webdata-vg/db-lv```

5. Re-mount the logical volume

```mount /dev/webdata-vg/db-lv /mnt/db```

### STEP 3: Install Wordpress on your Web Server EC2
1. Update the repository

```sudo yum -y update```

2. Install wget, Apache and it’s dependencies

```sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json```

3. Start Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd
```

4. To install PHP and it’s depemdencies

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

5. Restart Apache

```sudo systemctl restart httpd```

6. Download wordpress and copy wordpress to var/www/html
```
mkdir wordpress
cd wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
cp -R wordpress /var/www/html/
```

7. Configure SELinux Policies
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

### STEP 4: Install MySQL on your DB Server EC2
```
sudo yum update
sudo yum install mysql-server
```

Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

### STEP 5: Configure DB to work with Wordpress
1. Run the command:

```sudo mysql```

2. Create a database and name it 'wordpress'

```CREATE DATABASE wordpress;```

3. Create a username and a password of your choice. Remember to input your Web Server Private IP address in the code below before running it. You can do that on a notepad first.

```
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

### STEP 6: Configure Wordpress to work with remote Database
1. Open MySQL/Aurora port 3306 on DB Server EC2.
![](https://github.com/Abbysola/DevOps-Proj/blob/c8f747dbbf9de754d7ca7ba36cc33d10a81f86c9/Images/Proj6_Web%20Server%20Port%203306.png)

2. Install MySQL client on the Web Server and test that you can connect from your Web Server to your DB server by using mysql-client. Remember to input your DB Server Private IP address.
```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
3. Verify if you can successfully execute ```SHOW DATABASES;``` command and see a list of existing databases.

4. Edit the wp-config.php file by adding the database name 'wordpress', username, password and DB Server Private IP address.

```vi wp-config.php```

Type i to edit the file, then esc key, then :wq to save and quit.

![](https://github.com/Abbysola/DevOps-Proj/blob/c8f747dbbf9de754d7ca7ba36cc33d10a81f86c9/Images/Proj6_Editing%20wp-config.png)

5. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

6. Try to access from your browser the link to your WordPress 

```http://<Web-Server-Public-IP-Address>/wordpress/```

![](https://github.com/Abbysola/DevOps-Proj/blob/b50192b3e160e5a89b29e76153056e23d351ed9e/Images/Proj6_Wordpress1.png)

![](https://github.com/Abbysola/DevOps-Proj/blob/b50192b3e160e5a89b29e76153056e23d351ed9e/Images/Proj6_Wordpress2.png)

![](https://github.com/Abbysola/DevOps-Proj/blob/b50192b3e160e5a89b29e76153056e23d351ed9e/Images/Proj6_Wordpress3.png)











