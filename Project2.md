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

### CONFIGURING NGINX TO USE PHP PROCESSOR

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
*Save and close the file. Used 'nano' so I typed in CTRL+X and then y and ENTER to confirm.*

#### Activating the configuration by linking to the config file from Nginx’s sites-enabled directory:
```sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/```

*This will tell Nginx to use the configuration next time it is reloaded.*

#### Testing the configuration for syntax errors:
```sudo nginx -t```

#### The following output shows that there are no errors
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

#### DIsabling default Nginx host that is currently configured to listen on port 80:
```sudo unlink /etc/nginx/sites-enabled/default```

#### Reloading Nginx to apply changes
```sudo systemctl reload nginx```

*A new website is now active, but the web root /var/www/projectLEMP is still empty.*

#### Creating an index.html file in that location so that the new server block works as expected:
```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
#### Opening website URL using IP address on browser:
```http://<Public-IP-Address>:80```

*The image below shows the text from ‘echo’ command taht was written to index.html file. It means the Nginx site is working as expected. In the output, the server’s public hostname (DNS name) and public IP address are also seen.*

![]()

#### Running the command below to try it out using public hostname
```http://<Public-DNS-Name>:80```


---

### TESTING PHP WITH NGINX

---

#### Testing to validate that Nginx can correctly hand .php files off to your PHP processor. This was done by creating a test PHP file in the document root. Open a new file called info.php within the document root:
```sudo nano /var/www/projectLEMP/info.php```

#### Typing the following lines into the new file. This is valid PHP code that will return information about the server:
```
<?php
phpinfo();
```

#### Accessing this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php
```http://IP_address/info.php```

*This shows a web page containing detailed information about the server.*

![]()

#### Removing the file created as it contains sensitive information about the PHP environment 
```sudo rm /var/www/your_domain/info.php```

---
### RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)
---

#### Connecting to MySQL
```sudo mysql -p```

#### Creating a database
```mysql> CREATE DATABASE `example_database`;```

#### Creating a new user and granting him full privileges on the just created database. 'password' was replaced with secure password.
```mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';```

#### Giving the user permission over the example_database database
```mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';```

#### Exiting MySQL shell
```mysql> exit```

#### Testing if the new user has the proper permissions by logging in to the MySQL console using the custom user credentials:
```mysql -u example_user -p```

#### Confirming access to the example_database database:
```mysql> SHOW DATABASES;```

#### Output
```
SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| example_database   |
| information_schema |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)
```

#### Creating a Test Table named todo_list:
```
mysql> CREATE TABLE example_database.todo_list (
    -> item_id INT AUTO_INCREMENT,
    -> content VARCHAR(255),
    -> PRIMARY KEY(item_id)
    -> );
```

#### Inserting content into the test table
```mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");```
```mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");```
```mysql> INSERT INTO example_database.todo_list (content) VALUES ("My second important item");```
```mysql> INSERT INTO example_database.todo_list (content) VALUES ("My third important item");```
```mysql> INSERT INTO example_database.todo_list (content) VALUES ("My fourth important item");```

#### Confirming that the data was successfully saved:

```
Output
+---------+---------------------------+
| item_id | content                   |
+---------+---------------------------+
|       1 | My first important item   |
|       2 | My first important item   |
|       3 | My second important item  |
|       4 | My third  important item  |
|       5 | My fourth  important item |
+---------+---------------------------+
5 rows in set (0.03 sec)

mysql> exit
Bye

```

#### Exiting MySQL console
```mysql> exit```

#### Creating a PHP script that will connect to MySQL and query for the database content. 
```nano /var/www/projectLEMP/todo_list.php```

#### The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list.
#### Copying the content into the todo_list.php script where 'password' is replaced with the chosen password:
```
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```

#### Accessing the page in a web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php
```http://<IP_address>/todo_list.php```

*This shows the page below, showing the content inserted into the test table:*

![]()

*The PHP environment is ready to connect and interact with MySQL server.*





---
---
---

### Trouble shooting step 5
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




















