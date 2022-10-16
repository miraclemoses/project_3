# Simple To-Do application on MERN Web Stack
## MERN stack is a web development framework. It consists of MongoDB, ExpressJS, ReactJS, and NodeJS as its working components. Here are the details of what each of these components is used for in developing a web application when using MERN stack:
## MongoDB: A document-oriented, No-SQL database used to store the application data.
## NodeJS: The JavaScript runtime environment. It is used to run JavaScript on a machine rather than in a browser.
## ExpressJS: A framework layered on top of NodeJS, used to build the backend of a site using NodeJS functions and structures. Since NodeJS was not developed to make websites but rather run JavaScript on a machine, ExpressJS was developed.
## ReactJS: A library created by Facebook. It is used to build UI components that create the user interface of the single page web application.  

## As shown in the illustration above, the user interacts with the ReactJS UI components at the application front-end residing in the browser. This frontend is served by the application backend residing in a server, through ExpressJS running on top of NodeJS.
## Any interaction that causes a data change request is sent to the NodeJS based Express server, which grabs data from the MongoDB database if required, and returns the data to the frontend of the application, which is then presented to the user.
## This project demonstrates how to build a simple To-Do application on MERN Web Stack. 

__Prerequisites__
Ubuntu Server 20.04 LTS (this project is executed on a Microsoft Azure Ubuntu VM)
Putty (for Windows OS)
.ppk private key file to connect to the Ubuntu Server (The .ppk key is gotten from the conversion of the .pem file created when the Ubuntu VM was created. The .pem file is converted to .ppk using Puttygen).

# Step 1 - Backend configuration
Update Ubuntu
$ sudo apt update
Upgrade Ubuntu
$ sudo apt upgrade
Get the location of Node.js software from Ubuntu repositories
$ curl -sL https://deb.nodesource.com/setup_15.x | sudo -E bash -
Install Node.js on the server
$ sudo apt-get install -y nodejs
Note that the command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.
Verify the node installation with the command below
$ node -v 
Verify the node installation with the command below
$ npm -v

Application Code Setup
Create a new directory for the To-Do project
$ mkdir Todo
Run the command below to verify that the To-do directory is created with ls command
$ ls
Change the current directory to the newly created one:
$ cd Todo
Use the command npm init to initialize the project, so that a new file named package.json will be created. This file will contain information about the application and the dependencies that it needs to run. Follow the prompts after running the command. Press ‘Enter’ several times to accept default values, then accept to write out the package.json file by typing ‘yes’.


Run the command ls to confirm that the package.json file was created.

Install ExpressJS
To use express, install it using npm
$ npm install express
Create a file index.js with the command below
$ touch index.js
Run ls to confirm that your index.js file is successfully created
Install the dotenv module
$ npm install dotenv
Open the index.js file with a preferred text editor (Nano is used in this case)
$ sudo nano index.js
Copy the code below into it and save:

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



Notice that port 5000 has been specified to be used in the code. This will be required later when testing the configuration on the browser.
Save and exit the editor.
Start the server to see if it works by running this command in the same directory as the index.js file:
$ node index.js
‘Server running on port 5000’ would be seen in the terminal.

Add an inbound security rule to the Network Security Group of the VM to open inbound connection through port 5000:


Access the server’s Public IP or Public DNS name followed by port 5000 on a browser:
http://<PublicIP-or-PublicDNS>:5000


Routes
There are three actions that the To-Do application needs to be able to do:
Create a new task
Display list of all tasks
Delete a completed task
Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.
For each task, create routes that will define various endpoints that the To-do app will depend on. Create a folder named routes.
Open another Putty shell, connect to the Ubuntu VM and run the command:
$ cd Todo && mkdir routes
Change directory to routes folder
$ cd routes
Create a file api.js with the command below
$ touch api.js
Open the file with the command below
$ sudo nano api.js
Copy below code and paste in the file:

const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;


Models
Since the app is going to make use of Mongodb which is a NoSQL database, a model will be created.
A model is at the heart of JavaScript based applications, and it is what makes it interactive.
Use models to define the database schema. This is important as it is used to define the fields stored in each Mongodb document.
In essence, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties.
To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.
Change directory back to Todo folder with cd .. and install Mongoose
$ npm install mongoose
Create a new folder named models, change directory into the newly created models folder, and then create a file named todo.js
$ mkdir models && cd models && touch todo.js
Open the todo.js file and paste the code below in the file:
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

Update the routes from the file api.js in ‘routes’ directory to make use of the new model.
In Routes directory, open api.js with an editor, delete the code inside and paste the code below into it then save and exit:
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

MongoDB Database
A database is needed where to store data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS). Sign up for a shared clusters free account, which is ideal for this use case. Sign up here. Follow the sign up process, select Azure as the cloud provider, and choose a region near you.


Allow access to the MongoDB database from anywhere (Not secure, but it is ideal for testing)
IMPORTANT NOTE: In the image below, make sure the time of deleting the entry is changed from 6 Hours to 1 Week.

Create a MongoDB database and collection inside mLab




In the index.js file, process.env was specified to access environment variables, but this file has not been created.
Create a file in the Todo directory and name it .env.
$ touch .env
Note that this file is hidden once it is created and is not visible with ls but is visible with ls –a.
Open the file with an editor and add the connection string to access the database in it, just as below:
DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'
Ensure to update <username>, <password>, <network-address> and <database> according to the mlab setup. Here is how to get the connection string:




Update the index.js to reflect the use of .env so that Node.js can connect to the database.
Simply clear all the existing content in the index.js file, and update it with the entire code below:
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true })
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

Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.
Start the server using the command:
$ node index.js
A message ‘Database connected successfully’ is seen: 

The backend has been configured. The next step is to test it.

Testing Backend Code without Frontend using RESTful API
An API development client is needed to test the code using RESTful API.
In this project, Postman is used to test the API. Click Install Postman to download and install postman.
Click here to learn how to perform CRUD operations on Postman.
Test all the API endpoints and make sure they are working. For the endpoints that require body, send JSON back with the necessary fields since it’s what is set up in the code.
Open Postman and create a POST request to the API http://<PublicIP-or-PublicDNS>:5000/api/todos. This request sends a new task to the To-Do list so the application could store it in the database.
Make sure header key Content-Type as application/json is set.



Create a GET request to the API on http://<PublicIP-or-PublicDNS>:5000/api/todos. This request retrieves all existing records from the To-do application (backend requests these records from the database and sends it back as a response to GET request).


To delete a task, add its ID as a part of DELETE request:


Step 2 - Frontend creation
To start out with the frontend of the To-do app, the create-react-app command is used to scaffold the app.
In the same root directory as the backend code (To-Do directory), run:
$ npx create-react-app client

This will create a new folder in the To-do directory called client, where all the react code will be added.
Running a React App
Before testing the react app, there are some dependencies that need to be installed.
Install Concurrently. It is used to run more than one command simultaneously from the same terminal window.
$ npm install concurrently --save-dev
Install Nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
$ npm install nodemon --save-dev
In the Todo folder open the package.json file. Change the highlighted part of the below screenshot and replace with the code below.
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},



Configure Proxy in package.json
Change directory to ‘client’
$ cd client
Open the package.json file
$ sudo nano package.json
Add the key value pair in the package.json file "proxy": "http://localhost:5000"
The whole purpose of adding the proxy configuration above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos
Change directory back to the Todo directory, and simply run:
$ npm run dev
The app opens and is running on localhost:3000
In order to be able to access the application from the internet, open TCP port 3000 on the Ubuntu VM by adding an inbound security rule to the Network Security Group of the VM to open inbound connection through port 3000.


Creating the React Components
One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For the Todo app, there will be two stateful components and one stateless component. 
From your Todo directory run
$ cd client
Move to the src directory
$ cd src
Inside the src folder create another folder called components and cd into it
$ mkdir components && cd components
Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js.
$ touch Input.js ListTodo.js Todo.js
Open Input.js file and paste the following inside:
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

To make use of Axios, which is a Promise-based HTTP client for the browser and node.js, cd into your client from the terminal and run yarn add axios or npm install axios.
Move back to the client folder
$ cd ../..
Install Axios
$ npm install axios
Go to ‘components’ directory
$ cd src/components
Open the ListTodo.js
$ sudo nano ListTodo.js
In the ListTodo.js copy and paste the following code:
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

In the Todo.js file write in the following code:
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

Move to the src folder
$ cd ..
Make sure that you are in the src folder and open an existing App.js with an editor
$ sudo nano App.js
Clear the existing code in the file and paste the code below into it:
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

Open an existing file App.css with an editor 
$ sudo nano App.css
Clear the existing code in the file and paste the code below into it:
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

Open an existing file index.css with an editor 
$ sudo nano index.css
Clear the existing code in the file and paste the code below into it:
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

Go to the Todo directory
$ cd ../..
In the Todo directory run:
$ npm run dev

If there were no errors when saving all these files, the To-Do app is ready and fully functional with the functionality discussed earlier: creating a task, deleting a task and viewing all the tasks.



Conclusion
The MERN stack is a popular stack of technologies for building a modern single-page application.

Credits
https://www.educative.io/edpresso/what-is-mern-stack
http://makeseleniumeasy.com/2019/03/04/api-testing-tutorial-part-23-sending-delete-request-in-postman/
https://darey.io
