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

#### Checking that the pcakage.json file has been created
```ls```

### Installing Expressjs on the server

*Express is a framework for Node.js, therefore a lot of things developers would have programmed is already taken care of out of the box. It simplifies development, and abstracts a lot of low level details. For example, Express helps to define routes of your application based on HTTP methods and URLs.*

#### Installing Express using npm
```npm install express```

#### Creating a file named index.js
```touch index.js```

#### Checking that the file was created
```ls```

#### Installing the dotenv module
```npm install dotenv```
















