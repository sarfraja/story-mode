# Reading from MongoDB database

## What is this?
This is a tutorial and an example app which reads data from a MongoDB database. The app is using MongoDB, Express.js and Node.js.
I'm using Apple macOS High Sierra `10.13.4 (17E199)`, Node.js `v9.10.1`, npm `5.6.0`. 

This tutorial can be viewed on Medium as well, on [The Coding Hype](https://medium.com/thecodinghype/https-medium-com-thecodinghype-reading-from-mongodb-database-using-express-js-and-node-js-250ef8b9282a)

### Technologies and frameworks
MongoDB is a free and open-source cross-platform document-oriented NoSQL database program, which uses JSON-like documents with schemas.
Express.js is a minimalist web application framework or Node.js.
Node.js is an open-source, cross-platform JavaScript run-time environment that executes JavaScript code server-side. (Source: Wikipedia)

## Building the app

### MongoDB installation

First we need to install MongoDB. In this article I will install it by downloading community server from mongodb.com.
Let's extract the tgz file. I will use `~/mongodb` to store the mongodb runtime.

```
cd ~
tar -zxvf mongodb-osx-ssl-x86_64-3.6.3.tgz
mv mongodb-osx-ssl-x86_64-3.6.3 mongodb
```

MongoDB (`mongod`) defaults the database location to `/data/db/`.

Let's create this directory, and change its owner (`id -un` gets the username):

```
sudo mkdir -p /data/db
sudo chown -R `id -un` /data/db
```

Now we can start the MongoDB server (`mongod`):
```
cd ~
./mongodb/bin/mongod
```

### Data preparation
In a different shell window we need to start the MongoDB shell (`mongo`).
```
./mongodb/bin/mongo
```

In the MongoDB shell we can issue commands to the mongodb server to manipulate data in the database.

MongoDB stores data records in collections and the collections in databases. 
In this step we will create a new database called `tododb`. For this we will use the `use` statement.

```
use tododb
```

(The shell will return `switched to db tododb`.)

We will create a new collection todolist in this `tododb` database, and a new document in this todolist collection. The following command will create the collection as well as the new document.

```
db.todolist.insertOne({description: 'Register for the marathon', details: 'it must be done until 4.7.2018'})
```

The shell returns something like the following:

```
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5ac8fbc723f00907d3d1be99")
}
```

This means the document was created successfully. You can query this by the following command:
```
db.todolist.find()
```

Result:
```
{ "_id" : ObjectId("5ac8fbc723f00907d3d1be99"), "description" : "Register for the marathon", "details" : "it must be done until 4.7.2018" }
```

Or in a formatted way:
```
db.todolist.find().pretty()
```

Result:
```
{
	"_id" : ObjectId("5ac8fbc723f00907d3d1be99"),
	"description" : "Register for the marathon",
	"details" : "it must be done until 4.7.2018"
}
```

Let's add another entry to the `todolist`:

```
db.todolist.insertOne({description: 'Get money from ATM', details: '100 USD'})
```


### Express app creation

In this step we will create an express app. For this we need a new directory.
We initialize the content (actually the `package.json` file) with npm.

```
npm init -y
```

We need to install Express by issuing the following command. `--save` will save the dependency to package.json. We will also install `mongodb` which is the official MongoDB driver for Node.js.

```
npm install --save express mongodb
```

### Express server implementation: without db read

Create index.js:

```
touch index.js
```

The implementation of the server looks like the following. This needs to be put to `index.js`

```javascript
var express = require('express');
var app = express();

app.get('/', (req, res) => {res.send('Hello World!')})
app.listen(3000, () => console.log('Example app listening on port 3000!'))
```

The app can be run by issuing the following command:
```
node index.js
```

And then in the web browser enter `localhost:3000` to the URL.

### Express server implementation: implementing db read using mongodb node.js driver

For this we need to enhance index.js. The final code looks like the following. Please see the comments.

```javascript
// Import express
var express = require('express');
var app = express();

// Get MongoClient
var MongoClient = require('mongodb').MongoClient;

// db url, collection name and db name
const dburl = 'mongodb://localhost:27017';
const dbname = 'tododb';
const collname = 'todolist';

// process root url '/'
app.get('/', function(req, res) {

  // connect to DB
  MongoClient.connect(dburl, function(err, client) {
    if (!err) {

      // Get db
      const db = client.db(dbname);

      // Get collection
      const collection = db.collection(collname);

      // Find all documents in the collection
      collection.find({}).toArray(function(err, todos) {
        if (!err) {

          // write HTML output
          var output = '<html><header><title>Todo List from DB</title></header><body>';
          output += '<h1>TODO List retrieved from DB</h1>';
          output += '<table border="1"><tr><td><b>' + 'Description' + '</b></td><td><b>' + 'Details' + '</b></td></tr>';

          // process todo list
          todos.forEach(function(todo){
            output += '<tr><td>' + todo.description + '</td><td>' + todo.details + '</td></tr>';
          });

          // write HTML output (ending)
          output += '</table></body></html>'

          // send output back
          res.send(output);

          // log data to the console as well
          console.log(todos);
        }
      });

      // close db client
      client.close();
    }
  });
});

// listen on port 3000
app.listen(3000, function() {
  console.log('Example app listening on port 3000!')
});
```

The app can be started using

```
node index.js
```

## What are Future Plans for this Project?
Further development plans:
  * Implementing a REST API to query and change the db data
  * Using React to display the data
  * Enhance the app to be able to manupulate the data

