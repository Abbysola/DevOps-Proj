# DevOps Tooling Website Solution

### In this project, a solution that consists of following components will be implemented:
1. Infrastructure: AWS
1. Webserver Linux: Red Hat Enterprise Linux 8
1. Database Server: Ubuntu 20.04 + MySQL
1. Storage Server: Red Hat Enterprise Linux 8 + NFS Server
1. Programming Language: PHP
1. Code Repository: GitHub

Note: For this project, all the web servers will be created in the same subnet.

### Adding the AMI;

***Image

## Architectural Design

The diagram below shows a common pattern where several stateless Web Servers share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers, it look like a local file system from where they can serve the same files.

***Image

## Prepare the NFS Server

1. Create an EC2 instance with RHEL Linux 8 Operating System.
***Image

2. Configure 3 Logical Volumes on the Server. You can refer to ***Insert link*** to learn about how to create Logical volumes and Volume Groups.

Note that:

a. The disks will be formatted as xfs instead of ext4.

b. The 3 Logical Volumes will be named lv-opt lv-apps, and lv-logs.

c. Create mount points on /mnt directory for the logical volumes as follow:
Mount lv-apps on /mnt/apps – To be used by webservers
Mount lv-logs on /mnt/logs – To be used by webserver logs
Mount lv-opt on /mnt/opt – To be used by Jenkins server later

4. Install NFS server. It should be configured to start on reboot and should be up and running.

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

***Image

5.Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
```

Restart NFS Server ```sudo systemctl restart nfs-server.service```

Configure access to NFS for clients within the same subnet. Export the mounts for webservers’ subnet cidr to connect as clients. As said earlier, all three Web Servers are created inside the same subnet, but in production set up, it is best to separate each tier inside its own subnet for higher level of security.

To check your subnet cidr, open your EC2 instance in AWS web console and locate ‘Networking’ tab and open a Subnet link.

***Image

```sudo vi /etc/exports```

Insert the lines below (dont forget to replace with the Subnet CIDR):

```
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```
***Image

sudo exportfs -arv

6. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
```rpcinfo -p | grep nfs```

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

***Image

## Prepare the Web Servers

Create a RHEL EC2 instance on AWS which serves as our web server. Also remember to have in it in same subnet. In addition, open TCP Port 80 on the web server.

The following will be done on the web servers:

-Configure NFS client
-Deploy a Tooling application to our Web Servers into a shared NFS folder
-Configure the Web Servers to work with a single MySQL database

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

Make sure that the changes will persist on Web Server after reboot;
```sudo vi /etc/fstab```

add the line below (Insert the NFS server Private IP address);
```<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0```

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

setsebool -P httpd_execmem 1
```

6. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. To do this, create a new file on one server and check for the file on the other server;

```touch test.txt```

If you see the same files – it means NFS is mounted correctly. 

7. Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs.

Repeat step 4 to make sure the mount point will persist after reboot.

8. Fork the tooling source code from Darey.io Github Account to your Github account. )
Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html

Note: If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0
To make this change permanent – open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabledthen restrt httpd.

9. Update the website’s configuration to connect to the database (in /var/www/html/functions.php file). Apply tooling-db.sql script to your database using this command

```mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql```

10. Create in MySQL a new admin user with username: myuser and password: password:
    
```
INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);
```

Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and make sure you can login into the website with myuser















