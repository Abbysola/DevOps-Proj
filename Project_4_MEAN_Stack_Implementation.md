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

For this application, I'll add book records to MongoDB that contain book name, isbn number, author, and number of pages.

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
