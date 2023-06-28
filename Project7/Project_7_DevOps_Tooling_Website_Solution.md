# DevOps Tooling Website Solution

### In this project, a solution that consists of following components will be implemented:
1. Infrastructure: AWS
1. Webserver Linux: Red Hat Enterprise Linux 8
1. Database Server: Ubuntu 20.04 + MySQL
1. Storage Server: Red Hat Enterprise Linux 8 + NFS Server
1. Programming Language: PHP
1. Code Repository: GitHub

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


