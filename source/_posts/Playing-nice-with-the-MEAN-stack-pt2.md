---
title: Playing nice with the MEAN stack - part 2
tags:
  - MongoDB
  - Mongoose
  - AngularJS
  - Node
  - Express
date: 2017-04-04 23:22:39
---


![My favorite Taylor Swift song](/images/2017/04/meanstack.jpg)

*This is part 2 of a post on putting together a basic MEAN stack app. If you haven't read part one, check it out [here](/2017/03/21/Playing-nice-with-the-MEAN-stack-pt1/).*

In the [last post](/2017/03/21/Playing-nice-with-the-MEAN-stack-pt1/) we put together a very basic message board app using AngularJS, Node and Express. In this post, we're going to replace our temporary <code>mockDatabase</code> array an actual MongoDB database so that we can have persistent data storage. And we'll be using {% link Mongoose ODM http://mongoosejs.com/ %} to help us out.
<!-- more -->
<h3>Broad steps:</h3>

Broadly speaking, our approach is simple:
- Set up Mongoose connection, schema, and model
- Update our request handlers

If you're going to follow along, you'll need to make sure you've got MongoDB installed globally on your machine. And since we're going to be creating and accessing a database on our local machines, we'll need to run <code>mongod</code> in the terminal.

<h3>Some basic terminology</h3>

Before we get too far ahead of ourselves, let's go over some Mongoose/MongoDB terminology:

<h4><strong>Schema</strong></h4>

The blueprint: Schemas are essentially a definition of your documents and what info and methods they should contain. For example, if you want to have a document that describes a user, and you want to record specific information about each user, namely: username, first name, last name, and DoB. The schema is where you tell Mongoose the name and type (e.g. String, Boolean etc.) of information you want to store for any of your document types. Also, if your document will need any built-in methods, you can add them to the schema definition. Each schema automatically maps to a MongoDB collection.

{% codeblock Example %}
var userSchema = mongoose.schema({
  username: { type: String, required: true, unique },
  firstName: String,
  lastName: String,
  dob: Date
});
{% endcodeblock %}


<h4><strong>Model</strong></h4>

The constructor: A model is a constructor that is compiled from a schema. When you want to create a new document and save it. Models give you easy access to your collections so you can find documents, or update or remove a document.
Note: Since Mongoose models are constructors, the customary approach would be to capitalize the name of the variable when declaring e.g.
{% codeblock Example %}
var User = mongoose.model('User', userSchema);
{% endcodeblock %}

<h4><strong>Document</strong></h4>

The actual info: A document is an instance of a model. Documents can be created and saved to the database, then retrieved, updated or deleted. This is the format in which you store your actual data. So if you had 100 users, then you would have 100 user documents in your database.

<h4><strong>Collection</strong></h4>

MongoDB documents are grouped into collections. They are kind of like tables in MySQL, but they do not enforce a schema, and thus all documents within a collection don't need to have the same fields. This is what makes NoSQL database systems like MongoDB so flexible.

<h3>Let's do this!</h3>

<strong>Step 1. Connect to the database</strong>
In our <code>server.js</code> file, we're going to replace the <code>mockDatabase</code> with a real MongoDB database. Let's require Mongoose, set the address of the database and open a pending connection. I just called my database 'messages', and because I'm running it on my own machine, the address is <code>mongodb://localhost/messages</code>

{% codeblock server.js %}
var express = require('express');
var bodyParser = require('body-parser');
var morgan = require('morgan');
var mongoose = require('mongoose');

//Set address and open connection to database
mongoose.connect('mongodb://localhost/messages');
{% endcodeblock %}

Next up, we'll store that pending connection in a variable called <code>db</code> and then check whether the connection was successful or not:

{% codeblock server.js %}
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', _ => {
  console.log('database connected');
});
{% endcodeblock %}

Note that if the database doesn't exist, MongoDB will automatically create it when you try to insert a collection.

<strong>Step 2. Create a schema</strong>
Next up, we need to create a Mongoose schema for our message documents (remember, we're building a simple message board). We'll only need one field (called 'text') for our messages, and that should be of type <code>String</code>.

{% codeblock server.js %}
var messageSchema = new mongoose.Schema({
  text: String
});
{% endcodeblock %}

Note that some resources leave out the <code>new</code> keyword when calling mongoose.Schema(). You can use it or leave it out, but Mongoose will create a new instance either way. {% link See more on Stack Overflow. http://stackoverflow.com/questions/22596806/mongoose-schema-new-keyword %}

<strong>Step 3. Compile our schema into a model</strong>

{% codeblock server.js %}
var Message = mongoose.model('Message', messageSchema);
{% endcodeblock %}

<strong>Step 4. Update our request handlers</strong>
All we have left is to update our old request handlers (which were set up to work with the <code>mockDatabase</code> array). The GET request is super easy. We use Mongoose's <code>model.find</code> method, which takes a condition as its first argument. We'll just pass an empty object as the condition, which will give us back all messages in the collection. <code>find</code> is one of Mongoose's queries, which aren't exactly promises, but you can still follow it with a .then() block. (Note: if you need a fully-fledged promise, you can add use <code>exec</code> - {% link see the Mongoose docs for more http://mongoosejs.com/docs/promises.html %}).

Compare the old version with the updated request handler below:

{% codeblock server.js %}
// OLD GET REQUEST HANDLER
// app.get('/api/messages', (req, res) => {
//   res.send(mockDatabase);
// });

// UPDATED GET REQUEST HANDLER
app.get('/api/messages', (req, res) => {
  Messages.find({})
  .then(messages => {
    res.send(messages);
  })
});
{% endcodeblock %}

For our POST request, we create a new instance of a message object using our <code>Message</code> model. We <code>save</code> it, and then send the response back to the client confirming the text that we saved. See the comparison below:

{% codeblock server.js %}
// OLD POST REQUEST HANDLER
// app.post('/api/messages', (req, res) => {
//   mockDatabase.push({text: req.body.text});
//   res.send(`successfully posted: ${req.body.text}`);
// });

// UPDATED POST REQUEST HANDLER
app.post('/api/messages', (req, res) => {
  var newMessage = new Message({text: req.body.text})
  newMessage.save()
  .then(_ => {
    res.send(`message received and saved:  ${req.body.text}`);
  });
});
{% endcodeblock %}

So our final, updated <code>server.js</code> file should now look like this:

{% codeblock server.js %}
var express = require('express');
var bodyParser = require('body-parser');
var morgan = require('morgan');
var mongoose = require('mongoose');

//Set address and open connection to database
mongoose.connect('mongodb://localhost/messages');

//See if our pending connection (mongoose.connection) was successful or not
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', _ => {
  console.log('database connected');
});

// create schema
var messageSchema = mongoose.Schema({
  text: String
});

// compile this schema into a model
var Message = mongoose.model('Message', messageSchema);

// CREATE EXPRESS SERVER
var app = express();
// Some helpful middleware
app.use(bodyParser.json());
app.use(morgan('dev'));
app.use(express.static('../client'));

// Routes and request handlers
app.get('/api/messages', (req, res) => {
  Message.find({}).exec()
  .then(messages => {
    res.send(messages);
  });
});

app.post('/api/messages', (req, res) => {
  var newMessage = new Message({text: req.body.text})
  newMessage.save()
  .then(_ => {
    res.send(`message received and saved:  ${req.body.text}`);
  });
});

//Set server to listen
var port = 3000;
app.listen(port, () => {
  console.log(`server listening on ${port}`);
});
{% endcodeblock %}

And that's it! Obviously there are some refinements we could make, for example pulling out the routes, handlers, and schemas into separate modules to make our code a bit cleaner. But at least now you know how to connect a MongoDB database to your app!
