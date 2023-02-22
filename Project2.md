## LEMP ((Linux, Nginx, MySQL, PHP or Python, or Perl) STACK IMPLEMENTATION

#### Signed in to AWS free tier account and created a new EC2 Instance of t2.nano family with Ubuntu Server 20.04 LTS (HVM) image and connected into the instance. 

---

### INSTALLING THE NGINX WEB SERVER

---

#### Here, the NGINX web server is installed using the 'apt' package manager. Before that, the server's package index will be updated.

```sudo apt update```

#### Installing the Nginx package

```sudo apt install nginx```

*The Nginx is now running on Ubuntu 20.04 server.*

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

![NGINX Web Server Installation](https://github.com/Abbysola/DevOps-Proj/blob/0a1db7cfcdf46e8ddae8d4128fa0d734c585ce37/Images/Screenshot%202023-02-22%20at%2011.48.17.png)

*The image above shows that the web server is now correctly installed and accessible through your firewall. It is the same content that was gotten by ‘curl’ command, but represented in nice HTML formatting by on the web browser.*

---

### INSTALLING MYSQL

---

#### Installing MySQL, a Database Management System used within PHP environments. 'apt' will be used to acquire and install this software.

```sudo apt install mysql-server```

*Confirmed installation by typing Y, and then ENTER*

#### Logging into MySQL console

```sudo mysql```

*Using sudo to run this command will connect to the MySQL server as the administrative database user root.*

#### The output shows as seen below:

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.32-0ubuntu0.20.04.2 (Ubuntu)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
Query OK, 0 rows affected (0.07 sec)
```
#### Running a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to the database system. Before running the script, a password will be set for the root user, using mysql_native_password as default authentication method. This user’s password is defined as PassWord.1.

```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';```

#### Exiting the MySQL shell:

```mysql> exit```

#### The interactive script is started by running:

```sudo mysql_secure_installation```

#### Here, VALIDATE PASSWORD PLUGIN is configured. Answering Y for yes, or anything else to continue. Level 1 was ented for the level of password validation to avoid errors.

#### Testing that we can login to MySQL console with the password used after changing the root user password:

```sudo mysql -p```

### Exiting the MySQL console:

```mysql> exit```

---

### INSTALLING PHP

---

#### Here, PHP is installed to process code and generate dynamic content for the web server.

*While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. *

#### Installing php-fpm, which stands for “PHP fastCGI process manager”. Further, Nginx is told to pass PHP requests to this software for processing. Additionally, php-mysql is installed, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.*

#### Installing these 2 packages simultaneously:

```sudo apt install php-fpm php-mysql```

*When prompted, type Y and press ENTER to confirm installation.*

#### PHP components have now been installed.

---

### CONFIGURING NGINX TO USE PHP SERVER

---

*When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use projectLEMP as an example domain name.*

*On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the projectLEMP website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other site.*

#### Creating the root web directory for projectLEMP

```sudo mkdir /var/www/projectLEMP```

#### Assigning ownership of the directory with the $USER environment variable, which will reference the current system user

```sudo chown -R $USER:$USER /var/www/projectLEMP```

#### Opening a new configuration file in Nginx’s sites-available directory. Here, we’ll use nano:

```sudo nano /etc/nginx/sites-available/projectLEMP```

#### This will create a new blank file. Pasting the following bare-bones configuration:

```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
#### Save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.

### Activating the configuration by linking to the config file from Nginx’s sites-enabled directory:

```sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/```




*Trouble shooting*
#### Migrated to the folder/directory where the new configuration file was opened
```cd /etc/nginx/sites-available```

#### To check content
```ls```

#### To check the content of projectLEMP
```cat projectLEMP```

#### check PHP verion so it corresponds well on both ends and change if needed.
```php -v```

#### To edit content. Here, the php version was edited and server name was removed
```sudo vi projectLEMP```

```
server {
    listen 80;
    server_name _;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```

*Since we are using sudo, you close with*
1. esc
2. :wq
3. press ENTER

#### Restart nginx with:
```sudo systemctl restart nginx```

#### re-run the URL to check
```http://54.236.48.197/info.php```




















