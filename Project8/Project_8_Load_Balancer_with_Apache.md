# Load Balancer with Apache

In Project 7, I successfully configured 2 web servers and an NFS server. The 2 web servers will be used to configure the Load Balancer.

An approach used to cater for increased traffic is “horizontal scaling” – distributing load across multiple Web servers. This approach can be applied to any number for servers. This allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. 

In Project-7, both web servers configured had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of 2 servers, let alone millions of Google servers. To make this easy, a load balancer will be used to create a single point of access with a single public IP address. This helps to distribute clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

In this project, my Tooling Website solution will be enhanced using a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

### Architecture

![Architecture]()

### Prerequisites

The following servers must be installed and configured already (see Project 7 (https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Project_7_DevOps_Tooling_Website_Solution.md))

1. Two RHEL8 Web Servers
1. One MySQL DB Server (based on Ubuntu 20.04)
1. One RHEL8 NFS server

![Prerequisite_Architecture]()

### Configure Apache As A Load Balancer

1. Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb.

![EC2_Instance_List]()

2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

![TCP_Port80_for_LB]()

3. Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers.

Install apache2
```
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev
```

Enable the following modules
```
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```

Restart apache2 service
```sudo systemctl restart apache2```

Make sure apache2 is up and running by checking its status
```sudo systemctl status apache2```

Configure load balancing
```sudo vi /etc/apache2/sites-available/000-default.conf```

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>. Insert the Private IP addresses for both web servers

```
<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```

![Load_Balancing_configuration]()

Restart apache server
```sudo systemctl restart apache2```

Confirm that it is active
```sudo systemctl status apache2```

![Apache_Status]()

Here, 'bytraffic' balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by the loadfactor parameter.

There are other methods that can be used as well: bybusyness, byrequests, heartbeat.

To verify that my configuration works, i'll try to access my Load Balancer’s Public IP address or Public DNS name from my browser.

http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php


Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

Open two ssh/Putty consoles for both Web Servers and run following command:

sudo tail -f /var/log/httpd/access_log
Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.


