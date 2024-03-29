# Load Balancer with Apache

In Project 7, I successfully configured 2 web servers and an NFS server. The 2 web servers will be used to configure the Load Balancer.

An approach used to cater for increased traffic is “horizontal scaling” – distributing load across multiple Web servers. This approach can be applied to any number for servers. This allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. 

In Project-7, both web servers configured had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of 2 servers, let alone millions of Google servers. To make this easy, a load balancer will be used to create a single point of access with a single public IP address. This helps to distribute clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

In this project, my Tooling Website solution will be enhanced using a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

### Architecture

![Architecture](https://github.com/Abbysola/DevOps-Proj/blob/main/Project8/Images/1.Architecture.png)

### Prerequisites

The following servers must be installed and configured already (see Project 7 (https://github.com/Abbysola/DevOps-Proj/blob/main/Project7/Project_7_DevOps_Tooling_Website_Solution.md))

1. Two RHEL8 Web Servers
1. One MySQL DB Server (based on Ubuntu 20.04)
1. One RHEL8 NFS server

![Prerequisite_Architecture](https://github.com/Abbysola/DevOps-Proj/blob/main/Project8/Images/2.Prerequisite_architecture.png)

### Configure Apache As A Load Balancer

1. Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb.

![EC2_Instance_List](https://github.com/Abbysola/DevOps-Proj/blob/main/Project8/Images/3.EC2_Instance_list.png)

2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

![TCP_Port80_for_LB](https://github.com/Abbysola/DevOps-Proj/blob/main/Project8/Images/4.TCP_Port_80.png)

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

![Load_Balancing_configuration](https://github.com/Abbysola/DevOps-Proj/blob/main/Project8/Images/5.Load_Balancer_configuration.png)

Restart apache server
```sudo systemctl restart apache2```

Confirm that it is active
```sudo systemctl status apache2```

![Apache_Status](https://github.com/Abbysola/DevOps-Proj/blob/main/Project8/Images/6.Apache_restart.png)

Here, 'bytraffic' balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by the loadfactor parameter.

There are other methods that can be used as well: bybusyness, byrequests, heartbeat.

To verify that my configuration works, i'll try to access my Load Balancer’s Public IP address or Public DNS name from my browser.

http://Load-Balancer-Public-IP-Address-or-Public-DNS-Name/index.php

![LB_webpage](https://github.com/Abbysola/DevOps-Proj/blob/main/Project8/Images/7.LB_web_page.png)




