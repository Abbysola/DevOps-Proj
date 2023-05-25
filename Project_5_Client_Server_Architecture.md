## Client Server Architecture with MySQL

*Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another.*
*In their communication, each machine has its own role: the machine sending requests is usually referred as "Client" and the machine responding (serving) is called "Server".*

*In this project, the Web Server has a role of a "Client" that connects and reads/writes to/from a Database (DB) Server 
(MySQL), and the communication between them happens over a Local Network 
(It can also be Internet connection, but it is a common practice to place Web Server and DB Server close to each other in local network).*

### Practical Example of LAMP Website
#### Open up your Ubuntu or Windows terminal and run curl command

curl -Iv www.linkedin.com

#### I used the terminal on my Mac so I had 'curl' installed it by running:
```brew install curl```

*If your Ubuntu does not have ‘curl’, you can install it by running
```sudo apt install curl```*

*Here, my terminal will be the client, while www.hostloni.com will be the server.*

#### The response from the remote server is shown in below output. The requests from the URL are being served by a computer with an IP address 162.241.117.48 on port 80.

![Response from Remote Server]()

### IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).

#### Created and configured two Linux-based virtual servers (EC2 instances in AWS).

1. Server A - `mysql server`
1. Server B - `mysql client`

#### Install MySQL Server software on mysql server. To do this, connect to 'mysql server' instance.

#### Update and Upgrade the necessary packages

```sudo apt update```
```sudo apt upgrade -y```

#### Install mysql server by running the code below:

```sudo apt install mysql-server -y```

#### Install MySQL Client software on mysql server. To do this, connect to 'mysql client' instance.

#### Update and Upgrade the necessary packages

```sudo apt update```

#### Install mysql client software by running the code below:

```sudo apt install mysql-client```

#### Check to see that mysql client has been installed

```mysql --version```

*By default, both EC2 virtual servers are located in the same local virtual network, so they can communicate to each other
using local IP addresses. Mysql server's local IP address will be used to connect from mysql client. MySQL server uses TCP port 3306 by 
default, so it will be opened as a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not
allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.*

#### Opening Port 3306 in mysql server

![Port 3306]()

#### Configure MySQL server to allow connections from remote hosts

```sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf```

#### Replace ‘127.0.0.1’ to ‘0.0.0.0’ as seen below:
![Replacing Ports]()

#### From mysql client Linux Server, connect remotely to mysql server Database Engine without using SSH. 
#### Using Sql utility to perform this action:

#### Check to see that mysql is runnin on mysql server
```sudo systemctl status mysql```

#### Connect to mysql monitor on mysql server

```sudo mysql -u root```

#### Create a user on mysql server

```create user 'user'@'%' identified by 'password';```

*user and password can be replaced with any username and password of your choice. Here, I used user 'abby'.

#### On mysql client, connect to mysql server using the code below:

```mysql -h 'IP address of mysql server' -u 'username' -p```

#### My code looks like this:

```mysql -h 18.212.12.106 -u abby -p```

This will prompt you to enter your password and you are successfully connected.

#### Check that you have successfully connected to a remote MySQL server and can perform SQL queries:

```Show databases;```

#### A fully functional MySQL Client-Server set up has been deployed.




















