## MEAN (MongoDB Express Angular NodeJS) Stack Implementation
---
### STEP 1: Installation of NodeJS
---
### Update ubuntu
```sudo apt update```

### Upgrade ubuntu ###
```sudo apt upgrade```

### Add certificates ###

sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -

### Install NodeJS

sudo apt install -y nodejs

### STEP 2: Installation of Mongo DB

For this application, book records that contain book name, isbn number, author, and number of pages will bw added to MongoDB. 

```sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6```

```echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
### Install MongoDB

```sudo apt install -y mongodb```

### Start the server

sudo service mongodb start
