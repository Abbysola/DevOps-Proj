# DevOps Tooling Website Solution

### In this project, a solution that consists of following components will be implemented:
1. Infrastructure: AWS
1. Webserver Linux: Red Hat Enterprise Linux 8
1. Database Server: Ubuntu 20.04 + MySQL
1. Storage Server: Red Hat Enterprise Linux 8 + NFS Server
1. Programming Language: PHP
1. Code Repository: GitHub

Note: For this project, all the web servers will be created in the same subnet. For RHEL 8 server, use RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2.

### Adding the AMI;

![adding_amis](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/1.AMIs.png)

![public_images](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/2.Public_Images.png)

![launch_rhel_instance](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/3.Finding_RHEL.png)

## Architectural Design

The diagram below shows a common pattern where several stateless Web Servers share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers, it look like a local file system from where they can serve the same files.

![architecture](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/4.3-tier_Architecture.png)

## Prepare the NFS Server

1. Create an EC2 instance with RHEL Linux 8 Operating System.
![launch_instance](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/5.Launch_Instance.png)

2. Configure 3 Logical Volumes on the Server. You can refer to https://github.com/Abbysola/DevOps-Proj/blob/main/Project_6_Web_Solution_With_Wordpress.md to learn about how to create Logical volumes and Volume Groups.

Note that:

a. The disks will be formatted as xfs instead of ext4.

b. The 3 Logical Volumes will be named lv-opt lv-apps, and lv-logs.

c. Create mount points on /mnt directory for the logical volumes as follow:
Mount lv-apps on /mnt/apps – To be used by webservers
Mount lv-logs on /mnt/logs – To be used by webserver logs
Mount lv-opt on /mnt/opt – To be used by Jenkins server later

![creating_lvs_and_vgs](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/6.LVs_and_VGs.png)

3. Install NFS server. It should be configured to start on reboot and should be up and running.

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

![nfs_server_status](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/7.NFS_Server_Running.png)

4. Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
```

![setting-permissions](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/8.Setting_Permissions.png)

Restart NFS Server ```sudo systemctl restart nfs-server.service```

Configure access to NFS for clients within the same subnet. Export the mounts for webservers’ subnet cidr to connect as clients. As said earlier, all three Web Servers are created inside the same subnet, but in production set up, it is best to separate each tier inside its own subnet for higher level of security.

To check your subnet cidr, open your EC2 instance in AWS web console and locate ‘Networking’ tab and open a Subnet link.

![subnet](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/9.Subnet_link.png)

```sudo vi /etc/exports```

Insert the lines below (dont forget to replace with the Subnet CIDR):

```
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```

sudo exportfs -arv

5. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
```rpcinfo -p | grep nfs```

![see_info_on_ports](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/11.Info_on_req_ports.png)

Note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049

## Prepare the Database Server
Create an Ubuntu Server on AWS which will serve as our Database. Ensure its in the same subnet as the NFS-Server.

Here, we install and configure a MySQL DBMS to work with remote Web Server.

1. Install MySQL server
```
sudo apt -y update
sudo apt install -y mysql-server
sudo mysql
```

2. Create a database and name it tooling
3. Create a database user and name it webaccess
4. Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

![sql_database](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/10.Creating_SQL_Databases.png)

## Prepare the Web Servers

Create a RHEL EC2 instance on AWS which serves as our web server. Also remember to have in it in same subnet. In addition, open TCP Port 80 on the web server.

The following will be done on the web servers:

1. Configure NFS client
2. Deploy a Tooling application to our Web Servers into a shared NFS folder
3. Configure the Web Servers to work with a single MySQL database

### Steps
1. Launch a new EC2 instance with RHEL 8 Operating System
2. Install NFS client
   ```sudo yum install nfs-utils nfs4-acl-tools -y```

3. Mount /var/www/ and target the NFS server’s export for apps
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```

4. Verify that NFS was mounted successfully;
```df -h```

![mount_apps](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/12.Mounting_apps_on_var%3Awww.png)

Make sure that the changes will persist on Web Server after reboot;
```sudo vi /etc/fstab```

add the line below (Insert the NFS server Private IP address);

![adding_private_ip](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/13.Add_PrivateIP_to_etcfstab.png)

5. Install Remi’s repository, Apache and PHP
```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1
```

6. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. To do this, create a new file on one server and check for the file on the other server;

![creating_test_file](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/14.Creating_test_file_on_webserver.png)

If you see the same files – it means NFS is mounted correctly. 

![checking_test_file](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/15.Checking_test_file_on_NFS.png)

7. Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs.

![locating_log_folder](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/16.Test.txt_n_Webserverloghttpd.png)

Repeat step 4 to make sure the mount point will persist after reboot.

```sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd```

![mounting_logs](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/17.Mounting_logs_on_httpd.png)

```sudo vi /etc/fstab```

Add the line below:

```<NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd nfs defaults 0 0```

![editing_httpd_file](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/18.Editin_etc%3Afstab_for_logs.png)

8. Fork the tooling source code from Darey.io Github Account to your Github account. This is the source code used in this project.
Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html

Initialize git repository

Install git with ```sudo yum install git``` if it's not yet installed

```git init```

```git clone <tooling-webside-code-url```

(In this case, our tooling code is located at https://github.com/darey-io/tooling.git)
 
![forking_the_source_code_to_be_used](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/19.Cloning_the_folder_from_git.png)

Check for the html folder in both the respository and in /var/www/html. The html folder in /var/www/html is currently empty.

```cd tooling/```

```ls```

![checking_the_files](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/20.Check_files_in_tooling.png)

![checking_the_files](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/21.Check_files_in_%3Avar%3Awww.png)

Deploy the content of the html folder in the repository into /var/www/html

```sudo cp -R html/. /var/www/html```

![copying_the_folder_content](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/22.Copying_the_files.png)

Note: If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux 

```sudo setenforce 0```

To make this change permanent – open following config file 

```sudo vi /etc/sysconfig/selinux``` and set SELINUX=disabled then restart httpd.

![disabling_selinux](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/23.Disabling_sentenforce.png)

9. Update the website’s configuration to connect to the database (in /var/www/html/functions.php file).

![updating_db_configuration](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/24.Editing_db_credentials.png)

Change mysql bind address from this:

![changing_mysql_value](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/25.Changing_sql_value_for_db.png)

to this:

![changed_mysql_value](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/26.Changed_SQL_value_for_db.png)

Restart mysql

![restart_mysql](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/27.Restarting_mysql_on_db.png)

Add mysl security group on db

![adding_mysql_security_group](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/28.Security_group_for_mysql_on_db.png)

Apply tooling-db.sql script to your database using this command

```mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql```

![apply_tooling-db_script](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/29.Applying_tooling.db_script.png)

10. Create in MySQL a new admin user with username: myuser and password: password

Check users on tooling db

![check_users](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/30.Checking_sql_in_db.png)

Creating the new admin user
    
```
INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);
```

![creating_user](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/34.Creating_a_user.png)

Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and make sure you can login into the website with myuser user.

![landing_page](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/32.Landing_page.png)

![admin_home_page](https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Images/35.Admin_home_page.png)













