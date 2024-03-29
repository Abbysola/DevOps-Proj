## MEAN (MongoDB Express Angular NodeJS) Stack Implementation
---
### STEP 1: Installation of NodeJS
---
#### Update ubuntu
```sudo apt update```

#### Upgrade ubuntu
```sudo apt upgrade```

#### Add certificates

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

*Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.*

*Mongoose package which provides a straight-forward, schema-based solution to model the application data will be used. It will also be used to establish a schema for the database to store data of our book register.*

#### Install Express Mongoose
```sudo npm install express mongoose```

#### In the 'Books' folder, create a folder named 'apps' and navigate to the folder

```mkdir apps && cd apps```

#### Create a file named routes.js

```vi routes.js```

#### Copy and paste the code below into routes.js

```
const Book = require('./models/book');

module.exports = function(app){
  app.get('/book', function(req, res){
    Book.find({}).then(result => {
      res.json(result);
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while retrieving books');
    });
  });

  app.post('/book', function(req, res){
    const book = new Book({
      name: req.body.name,
      isbn: req.body.isbn,
      author: req.body.author,
      pages: req.body.pages
    });
    book.save().then(result => {
      res.json({
        message: "Successfully added book",
        book: result
      });
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while saving the book');
    });
  });

  app.delete("/book/:isbn", function(req, res){
    Book.findOneAndRemove(req.query).then(result => {
      res.json({
        message: "Successfully deleted the book",
        book: result
      });
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while deleting the book');
    });
  });

  const path = require('path');
  app.get('*', function(req, res){
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
  });
};
```

#### In the ‘apps’ folder, create a folder named 'models' and navigate to the folder

```mkdir models && cd models```

#### Create a file named book.js

```vi book.js```

#### Copy and paste the code below into ‘book.js’

```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```

### Step 3: Access the routes with AngularJS
*AngularJS provides a web framework for creating dynamic views in your web applications. AngularJS will be used to connect our web page with Express and perform actions on our book register.*

#### Change the directory back to ‘Books’

```cd ../..```

#### Create a folder named 'public' and navigate to the folder

```mkdir public && cd public```

#### Add a file named script.js

```
vi script.js
Copy and paste the Code below (controller configuration defined) into the script.js file.

var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```

#### In public folder, create a file named 'index.html'

```
vi index.html
Cpoy and paste the code below into index.html file.

<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```

#### Change the directory back to Books

```cd ..```

#### Start the server

```node server.js```

The server is now up and running. It can be connected via port 3300. For this, TCP port 3300 will be opened in the AWS Web Console for the EC2 Instance.

![Opening Port 3300](https://github.com/Abbysola/DevOps-Proj/blob/1172bc074aabb836f129a05728b6e370cd81d56f/Images/Port%203300.png)

#### A separate Putty or SSH console can be run to test what curl command returns locally.

curl -s http://localhost:3300

*It shall return an HTML page, it is hardly readable in the CLI. However, It can also be acceseed from the Internet.*

#### The Book Register web application can be accessed from the Internet with a browser using Public IP address or Public DNS name.

http://34.230.81.169:3300

![Web book register application](https://github.com/Abbysola/DevOps-Proj/blob/9132d2f0ddd3230ec4a744935254b74fd7d3af1c/Images/Web%20Book%20Register%20Application.png)

![Web book register application 2](https://github.com/Abbysola/DevOps-Proj/blob/e633c297de3ee218d6a59f8a20781eca80d46bfd/Images/Web%20Book%20Register%20Application2.png)

*How to get your server’s Public IP and public DNS name:*

1. It can be found in your AWS web console in EC2 details
2. Run curl -s http://169.254.169.254/latest/meta-data/public-ipv4 for Public IP address or curl -s http://169.254.169.254/latest/meta-data/public-hostname for Public DNS name.

This is how the Web Book Register Application looks like in a browser







