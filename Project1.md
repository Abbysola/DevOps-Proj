## LAMP (Linux, Apache, MySQL, PHP or Python, or Perl) STACK IMPLEMENTATION ##
---
##### Setting up AWS account and connecting to my EC2 instance from my terminal. Downloaded the PEM file and navigated to the folder it has been saved. #####
*The anchor tags < > represent specific values relating to different situations*

````cd <PEM-file-location>````

##### Changing permissions of the private key file: #####

```sudo chmod 0400 <private-key-name>.pem```

##### Connecting to the instance: #####

```ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>```

##### This created a Linux server in the cloud #####

### INSTALLING APACHE AND UPDATING THE FIREWALL ###

##### Installing Apache using Ubuntu's package manager 'apt' #####

##### Updating a list of packages and running apache2 package installation #####

```
sudo apt update

sudo apt install apache2
```
##### To verify that apache2 is running as a service in the OS #####

```sudo systemctl status apache2```

*This launches a webserver in the clouds. The server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).*

#### Accessing the Apache HTTP server. This can be done from the terminal or we can also see if it responds to requests from the Internet. ####

##### From the terminal: #####

```
curl http://localhost:80
or
 curl http://127.0.0.1:80
 ```
##### From the Internet. Opening a web browser and accessing the following url: #####

http://<Public-IP-Address>:80

*The Public IP address can be checked from the AWS console or by typing the following command in the terminal:*

curl -s http://169.254.169.254/latest/meta-data/public-ipv4
