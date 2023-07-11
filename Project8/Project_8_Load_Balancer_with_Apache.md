# Load Balancer with Apache

In Project 7, I successfully configured 2 web servers and an NFS server. The 2 web servers will be used to configure the Load Balancer.

An approach used to cater for increased traffic is “horizontal scaling” – distributing load across multiple Web servers. This approach can be applied to any number for servers. This allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. 

In Project-7, both web servers configured had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of 2 servers, let alone millions of Google servers. To make this easy, I used a load balancer to create a single point of access with a single public IP address. this helps to distribute clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

### Architecture

![Architecture](/Users/abisolaajuwon/Desktop/DareyPractice/DevOps-Proj/Project8/Images/1.Architecture.png)