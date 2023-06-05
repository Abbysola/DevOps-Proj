### STEP 1: Prepare a Web Server
1. Launch an EC2 instance to serve as the web server. Create 3 volumes in the same zone as the Web Server, each of 10 gig
1. Attach all three volumes one by one to the Web Server.
1. Open the Linux terminal to begin configuration. Use the ```lsblk``` command to inspect what block devices are attached to the server. Notice the names of your newly created devices. All devices in Linux reside in /dev/directory. Inspect it with ```ls /dev/``` and make sure you see all 3 newly created block devices there – their names will be xvdf, xvdh, xvdg.
1. Use ```df -h``` command to see all mounts and free space on your server.
1. Use 'gdisk' utility to create a single partition on each of the 3 disks.
```sudo gdisk /dev/xvdf```

*Repeat this for the other disks - xvdh and xvdg*
1. Install 'lvm2' package using ```sudo yum install lvm2```. Run ```sudo lvmdiskscan``` command to check for available partitions.
Note: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
