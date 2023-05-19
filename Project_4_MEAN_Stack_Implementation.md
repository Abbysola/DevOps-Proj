## MEAN (MongoDB Express Angular NodeJS) Stack Implementation
---
### STEP 1: Installation of NodeJS
---
#### Update ubuntu
```sudo apt update```

#### Upgrade ubuntu ###
```sudo apt upgrade```

#### Add certificates ###

sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -

#### Install NodeJS

sudo apt install -y nodejs

### STEP 2: Installation of Mongo DB

For this application, book records that contain book name, isbn number, author, and number of pages will bw added to MongoDB. 

```sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6```

```echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
#### Install MongoDB

```sudo apt install -y mongodb```

#### Start the server

```sudo service mongodb start```

#### Check that the service is up and running

```sudo systemctl status mongodb```

#### Install npm – Node package manager

```sudo apt install -y npm```

#### Install a body-parser package

The ‘body-parser’ package will help to process JSON files that are passed in requests to the server

```sudo npm install body-parser```

#### Create a folder named ‘Books’

```mkdir Books && cd Books```

#### In the Books directory, initialize npm project

```npm init```

#### Add a file to it named server.js

```vi server.js```

#### Copy and paste the web server code below into the server.js file

```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```

### Step 3: Install Express and set up routes to the server

*Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.

*We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.


