## MERN (MongoDB, ExpressJS, ReactJS, Node.js) STACK IMPLEMENTATION
---
#### The task for this project is to create a simple to-do application on MERN web stack in AWS cloud

---
### BACKEND CONFIGURATION
---

#### Connecting to an EC2 Instance on AWS with the usual commands

#### Updating Ubuntu
```sudo apt update```

#### Upgrading Ubuntu
```sudo apt upgrade```

#### Getting the location of Node.js software from Ubuntu repositories
```curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -```

### Installing Node.js on the server

#### Installing node.js on the server with the command below. This command will install both nodejs and npm. 
```sudo apt-get install -y nodejs```

*NPM is a package manager for Node like apt for Ubuntu. It is used to install Node modules & packages and to manage dependency conflicts.

#### Verify the node and npm installations with the commands:
```node -v```
```npm -v```

### Application Code Set-Up

#### Creating a new directory for the To-Do project:
```mkdir Todo```

#### Verifying that the directory was created
```ls```

*ls -lih will give more information about the directory. ls --help shpws the different useful keys with ls*

#### Changing the current directory to the newly created one

```cd Todo```

#### Using the command below to initialise the project, so that a new file named package.json will be created. This file will normally contain information about the application and the dependencies that it needs to run. Press Enter several times to accept default values, then accept to write out the package.json file by typing yes.
```npm init```

![Creating package.json file](https://github.com/Abbysola/DevOps-Proj/blob/70ac82ad681469ef9b66964969c668fcfc6e9135/Images/Screenshot%202023-02-25%20at%2016.08.11.png)

#### Checking that the pcakage.json file has been created
```ls```

---
### INSTALLING EXPRESSJS
---

*Express is a framework for Node.js, therefore a lot of things developers would have programmed is already taken care of out of the box. It simplifies development, and abstracts a lot of low level details. For example, Express helps to define routes of your application based on HTTP methods and URLs.*

#### Installing Express using npm
```npm install express```

#### Creating a file named index.js
```touch index.js```

#### Checking that the file was created
```ls```

#### Installing the dotenv module
```npm install dotenv```

#### Opening the index.js file
```vim index.js```

#### Pasting the code below into the file
```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```

*Use :w to save in vim and use :qa to exit vim.*

#### Starting the server to see if it works. Running the command below in the same directory where index.js file was stored.
```node index.js```

#### The image below shows the server running on port 5000
![Server Running on Port 5000](https://github.com/Abbysola/DevOps-Proj/blob/70ac82ad681469ef9b66964969c668fcfc6e9135/Images/Screenshot%202023-02-25%20at%2016.29.22.png)

#### Create an inbound rule to open port 5000

*Server’s Public IP and public DNS name can be gotten either from the AWS web console in EC2 details or by running curl -s http://169.254.169.254/latest/meta-data/public-ipv4 for Public IP address or curl -s http://169.254.169.254/latest/meta-data/public-hostname for Public DNS name.*

#### Accessing the server through port 5000
```http://<100.25.44.38/>:5000``` 
OR
```http://ec2-100-25-44-38.compute-1.amazonaws.com:5000```

#### The image below is seen
![Welcome to Express](https://github.com/Abbysola/DevOps-Proj/blob/70ac82ad681469ef9b66964969c668fcfc6e9135/Images/Screenshot%202023-02-25%20at%2016.43.01.png)

### Routes
The To-Do application needs to be able to:
1. Create a new task
1. Display list of all tasks
1. Delete a completed task

*Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE. For each task, routes will be created that will define various endpoints that the To-do app will depend on. *

#### Create a folder 'routes'
```mkdir routes```

#### Changin the directory to routes folder
```cd routes```

#### Create a api.js file
```touch api.js```

#### Opening the file
```vim api.js```

#### Pasting the code below into the file

```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

---
### MODELS
---
*Since the app is going to make use of Mongodb which is a NoSQL database, a model will be created. A model is at the heart of JavaScript based applications, and it is what makes it interactive. Models will also be used to define the database schema. This will help to define the fields stored in each Mongodb document.*

*The Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties.*

*To create a Schema and a model, mongoose which is a Node.js package that makes working with mongodb easier is installed*

#### Changing directory back to Todo folder with cd .. and installing Mongoose
```npm install mongoose```

#### Creating a new folder 'models':
```mkdir models```

#### Change directory into the newly created folder models
```cd models```

#### Create a file named todo.js
```touch todo.js```

*All three commands above can be defined in one line to be executed consequently with help of && operator:*
```mkdir models && cd models && touch todo.js```

#### Opening vim todo.js fle and pasting the code below
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```

#### Updating our routes from the file api.js in ‘routes’ directory to make use of the new model. To do this,the api.js file is opened with:
```vim api.js```

*Delete the code inside with :%d command

#### Pasting the code below into it then save with :w and exit with :qa
```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```

---
### MONGODB DATABASE
---

*We need a database where we will store our data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS), so to make life easy, we will need to sign up for a shared clusters free account, which is ideal for our use case. Follow the sign up process, select AWS as the cloud provider, and choose a region near you.*




























