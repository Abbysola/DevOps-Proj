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

1. Create a MongoDB cluster and allow access to it from anywhere (not secure but this is just for testing)
1. Under the Network Access tab, Change the time of deleting the entry from 6 Hours to 1 Week
1. Under the Cluster's tab (Atlas), Create a MongoDB database and collection inside mLab
1. Select 'Add My Own Data'

#### In the index.js file, process.env was specified to access environment variables, but the file has not been created.

#### Creating a file in your Todo directory and name it .env.
```
touch .env
vi .env
```

#### Adding the connection string to access the database in it, just as below:
```DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'```

#### where username, password, network-address and database are updated with the credentials used.

*The connection string can be gotten by connecting to the cluster then selecting 'connect your application'. Make sure the DRIVER is valued at node.js.*

#### Updating the index.js to reflect the use of .env so that Node.js can connect to the database. The existing content in the file is deleted, and updated it with the code below.

#### Opening the file

```vim index.js```

#### To do this:
1. Press esc
1. Type :%d to delete the content of index.js
1. Hit ‘Enter’
1. Press i to enter the insert mode in vim
1. Paste the entire code below in the file
1. Type :w to save the file
1. Type :qa to quit

```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```

*Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

#### Starting the server
```index node.js```

*A message ‘Database connected successfully’ shows that the backend is configured.*

##### TESTING BACKEND CODE WITHOUT FRONTEND USING RESTFUL API

#### Here, ReactJS will be used to create a Frontend UI and Postman will be used to test the API.

#### Open Postman, create a POST request to the API 
```http://<PublicIP-or-PublicDNS>:5000/api/todos``` 

#### This request sends a new task to our To-Do list so the application could store it in the database.

#### Set header key Content-Type as application/json

#### Post a request to the API. Also, create a GET request to the API on:
```http://<PublicIP-or-PublicDNS>:5000/api/todos```

*This request retrieves all existing records from out To-do application (backend requests these records from the database and sends it us back as a response to GET request).*

### FRONTEND CREATION

#### Creating a user interface for a Web client (browser) to interact with the application via API. To start out with the frontend of the To-do app, the create-react-app command to structure the app.

#### In the To-do directory, run:
```npx create-react-app client```

*This will create a new folder in your Todo directory called client, where you will add all the react code

##### RUNNING A REACT APP

#### Before testing the react app, there are some dependencies that need to be installed.

1. Installing concurrently. This is used to run more than one command simultaneously from the same terminal window.
```npm install concurrently --save-dev```
1. Install nodemon. This is used to run and monitor the server. If there is any change in the server code, Nodemon helps to restart the server automatically if there is any change in the server code and load the new changes.
```npm install nodemon --save-dev```
1. The package.json file is opened from the Todo folder. Replace the content part of "scripts" in the file with the code below:
```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```

#### Configuring Proxy in package.json
1. Change the directory to the client folder
1. Open the package.json file
```vi package.json```
1. Add the key value pair in the package.json file
```"proxy": "http://localhost:5000"```
*The proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos

#### Run the code below in the Todo directory
```npm run dev```

*The app should open and start running on localhost:5000*

##### CREATING REACT COMPONENTS
#### Running the code below from the Todo directory
```cd client/src```

#### Creating another folder called 'components' in the src folder
```mkdir components```

#### Moving into the components directory
```cd components```

#### Creating files inside the components directory
```touch Input.js ListTodo.js Todo.js```

#### Opening input.js file
```vi Input.js```

#### Pasting the code below
```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```

*To make use of Axios, which is a Promise based HTTP client for the browser and node.js, cd into the client from the terminal and run yarn add axios or npm install axios.*

#### Move to the clients folder
```cd ../..```

#### Install Axios
```npm install axios```

#### Moving to the components directory
```cd src/components```

#### Opening ListTodo.js
```vi ListTodo.js```

#### Copy and Paste the code below
```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo
```

#### Write the code below in the Todo.js file
```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```

#### Editing the App.js file to the code below

#### Moving to the src folder and run 'vi App.js'
```cd ..```
```vi App.js```

```
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
```

#### Running the below in the src directory:
```vi App.css```

#### Pasting the code below:
```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```

#### Running the below in the src directory:
```vim index.css```

#### Pasting the code below:
```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```

#### Navigating to the Todo directory:
```cd ../..```

#### Running the code below:
```npm run dev```































