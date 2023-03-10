## LAMP (Linux, Apache, MySQL, PHP or Python, or Perl) STACK IMPLEMENTATION

---

*Kindly note that images will be added later. The project was first done without making screenshots so I couldnt make screenshots when I eventually started documenting because I could not get those images again. Thanks. *

##### Setting up AWS account and connecting to my EC2 instance from my terminal. Downloading the PEM file and navigated to the folder it has been saved.

*The anchor tags < > represent specific values relating to different situations*

````cd <PEM-file-location>````

##### Changing permissions of the private key file:

```sudo chmod 0400 <private-key-name>.pem```

##### Connecting to the instance:

```ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>```

##### This creates a Linux server in the cloud

---

### INSTALLING APACHE AND UPDATING THE FIREWALL

---

##### Installing Apache using Ubuntu's package manager 'apt'

##### Updating a list of packages and ran apache2 package installation

```
sudo apt update

sudo apt install apache2
```
##### To verify that apache2 is running as a service in the OS

```sudo systemctl status apache2```

*This launches a webserver in the clouds. The server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).*

#### Accessing the Apache HTTP server. This was done from the terminal and also checked if it responds to requests from the Internet.

##### From the terminal:

```
curl http://localhost:80
or
 curl http://127.0.0.1:80
 ```
##### From the Internet. Opening a web browser and accessing the following url:

```http://<Public-IP-Address>:80```

*The Public IP address can be checked from the AWS console or by typing the following command on the terminal:*

```curl -s http://169.254.169.254/latest/meta-data/public-ipv4```

---

### INSTALLING MYSQL

---

##### Installing a Database Management System (DBMS) which is MySQL to store and manage data. This was done using 'apt'

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

### CREATING A VIRTUAL HOST FOR A WEBSITE USING APACHE

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
*To save and close the file, simply follow the steps below:*

*-Hit the esc button on the keyboard*\
*-Type :*\
*-Type wq. w for write and q for quit*\
*-Hit ENTER to save the file*

##### 'ls' command shows the new file in the sites-available directory
```sudo ls /etc/apache2/sites-available```

##### Using a2ensite command to enable the new virtual host:

```sudo a2ensite projectlamp```

##### Disabling the default website that comes installed with Apache. Disabling Apache’s default website use a2dissite command:

```sudo a2dissite 000-default```

##### To make sure the configuration does not contain syntax errors:

```sudo apache2ctl configtest```

##### Reloading so that the changes can take effect

```sudo systemctl reload apache2```

*Now the website is active*

##### Creating an index.html file in root /var/www/projectlamp which s still empty so that the virtual host works as expected:

```sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html```

##### Opening the website using IP address

```http://<Public-IP-Address>:80```

*The text from the ‘echo’ command written to the index.html file shoss here and this means the Apache virtual host is working as expected. The server’s public hostname (DNS name) and public IP address is also seen.*

##### The website can also be accessed in t the owser by public DNS name (port is optional)

```http://<Public-DNS-Name>:80```

---

### ENABLE PHP ON THE WEBSITE

---


##### To stop the index.html file from overriding the index.php file, we run the command below and edit the /etc/apache2/mods-enabled/dir.conf file, Also, the order in which the index.php file is listed withing the DirectoryIndex directive is changed:

```sudo vim /etc/apache2/mods-enabled/dir.conf```

```
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

##### Reloading the Apache so the changes can take effect

```sudo systemctl reload apache2```

##### Creating a PHP script to test that PHP is correctly installed and configured on your server.

##### Now that there is a custom location to host your website’s files and folders, a PHP test script is created to confirm that Apache is able to handle and process requests for PHP files.

##### Creating a new file named index.php inside the custom web root folder:

```vim /var/www/projectlamp/index.php```

##### This opens a blank file. Adding the following text, which is valid PHP code, inside the file:

```
<?php
phpinfo();
```

*After checking the relevant information about the PHP server created through that page, the file will be removed as it contains sensitive information about the PHP environment.*

```sudo rm /var/www/projectlamp/index.php```
















 












