## LAMP (Linux, Apache, MySQL, PHP or Python, or Perl) STACK IMPLEMENTATION ##
---
##### Set up AWS account and connecting to my EC2 instance from my terminal. Downloaded the PEM file and navigated to the folder it has been saved. #####
*The anchor tags < > represent specific values relating to different situations*

````cd <PEM-file-location>````

##### Changed permissions of the private key file: #####

```sudo chmod 0400 <private-key-name>.pem```

##### Connected to the instance: #####

```ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>```

##### This created a Linux server in the cloud #####

---

### INSTALLING APACHE AND UPDATING THE FIREWALL ###

---

##### Installed Apache using Ubuntu's package manager 'apt' #####

##### Updated a list of packages and ran apache2 package installation #####

```
sudo apt update

sudo apt install apache2
```
##### To verify that apache2 is running as a service in the OS #####

```sudo systemctl status apache2```

*This launches a webserver in the clouds. The server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).*

#### Accessed the Apache HTTP server. This was done from the terminal and also checked if it responds to requests from the Internet. ####

##### From the terminal: #####

```
curl http://localhost:80
or
 curl http://127.0.0.1:80
 ```
##### From the Internet. Opened a web browser and accessing the following url: #####

```http://<Public-IP-Address>:80```

*The Public IP address can be checked from the AWS console or by typing the following command on the terminal:*

```curl -s http://169.254.169.254/latest/meta-data/public-ipv4```

---

### INSTALLING MYSQL ###

---

##### Installed a Database Management System (DBMS) which is MySQL to store and manage data. This was done using 'apt' #####

```sudo apt install mysql-server```

##### Ran an interactive script to remove some insecure default settings and lock down access to the database system. Before running the script, a password was set a password for the root user, using mysql_native_password as default authentication method.

```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY <Password>';```

##### The interactive script is run with this:

```sudo mysql_secure_installation```

##### Answer Yes (y or Y) to continue at each point and set a new password. After this, I logges in to MySQL console using:

```sudo mysql -p```

##### Exited the MySQL console using:

```mysql> exit```

---

### INSTALLING PHP ###

---

##### The Packages that will be installed are:
##### 1. PHP: the component of our setup that will process code to display dynamic content to the end user. 
##### 2. Php-mysql: a PHP module that allows PHP to communicate with MySQL-based databases.
##### 3. Libapache2-mod-php: will be used to enable Apache to handle PHP files. 
##### Core PHP packages will automatically be installed as dependencies.

##### Installed the packages using:

```sudo apt install php libapache2-mod-php php-mysql```

##### To confirm the PHP version:

```php -v```

```
PHP 8.1.2-1ubuntu2.10 (cli) (built: Jan 16 2023 15:19:49) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.2, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.2-1ubuntu2.10, Copyright (c), by Zend Technologies
```

---

### CREATING A VIRTUAL HOST FOR A WEBSITE USING APACHE ###

---

##### A domain called 'projectlamp' was set-up here.
##### Apache on Ubuntu 20.04 has one server block enabled by default that is configured to serve documents from the /var/www/html directory.

##### Created the directory for 'projectlamp' by adding it to the default one using ‘mkdir’ command as follows:

```sudo mkdir /var/www/projectlamp```

##### Assigned the owner of the directory with the current system user

```sudo chown -R $USER:$USER /var/www/projectlamp```

##### Created and opened a new configuration file in Apache’s sites-available directory

```sudo vi /etc/apache2/sites-available/projectlamp.conf```

##### This creates a new blank file. Pasted the text below in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode:

```
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
*To save and close the file, simply follow the steps below:

*-Hit the esc button on the keyboard
-Type :
-Type wq. w for write and q for quit
-Hit ENTER to save the file

##### 'ls' command shows the new file in the sites-available directory
```sudo ls /etc/apache2/sites-available```

























