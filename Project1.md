## LAMP (Linux, Apache, MySQL, PHP or Python, or Perl) STACK IMPLEMENTATION ##
---
#### Set up AWS account and connected to my EC2 instance from my terminal. Downloaded the PEM file and navigated to the folder it has been saved. ####
#### The anchor tags < > represent specific values relating to different situations ####

````cd <PEM-file-location>````

#### Change permissions of the private key file ####

```sudo chmod 0400 <private-key-name>.pem```

#### Connect to the instance ####

```ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>```

```sudo apt update```\
```sudo apt install apache2```
