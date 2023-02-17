## LEMP ((Linux, Nginx, MySQL, PHP or Python, or Perl) STACK IMPLEMENTATION

#### Signed in to AWS free tier account and created a new EC2 Instance of t2.nano family with Ubuntu Server 22.04 LTS (HVM) image and connected into the instance. 

---

### INSTALLING THE NGINX WEB SERVER

---

#### Here, the NGINX web server is installed using the 'apt' package manager. Before that, the server's package index will be updated.

```sudo apt update```

#### Installing the Nginx package

```sudo apt install nginx```

*The Nginx is now running on Ubuntu 22.04 server.*

#### To verify that nginx was successfully installed and is running as a service in Ubuntu, run:

```sudo systemctl status nginx```

#### In order to reieve traffic from the Web Server, TCP port 80 which is default port that web brousers use to access web pages in the Internet will be opened. TCP port 22 is open by default on the EC2 machine and to access it via SSH, a rule will be added to the EC2 configuration to open inbound connection through port 80.

#### Accessing the server locally:

```
curl http://localhost:80
or
curl http://127.0.0.1:80
```

*These two commands above produce the same result – they use ‘curl’ command to request our Nginx on port 80. The difference is that: In the first case, the server is accessed via DNS name and in the second one – by IP address (in this case IP address 127.0.0.1 corresponds to DNS name ‘localhost’ and the process of converting a DNS name to IP address is called "resolution").*

#### Checking how the Nginx server responds to requests from the Internet:

*Try the command below on any web browser. The URL also works without specifying port number since all web browsers use port 80 by default.*

```http://<Public-IP-Address>:80```

#### Another way to retrieve your Public IP address, other than to check it in AWS Web console, is to use following command:

```curl -s http://169.254.169.254/latest/meta-data/public-ipv4```

*The image above shows that the web server is now correctly installed and accessible through your firewall. It is the same content that was gotten by ‘curl’ command, but represented in nice HTML formatting by on the web browser.*

---

### INSTALLING MYSQL

---

#### Installing MySQL, a Database Management System used within PHP environments. 'apt' will be used to acqire and install this software.











