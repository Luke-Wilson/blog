---
title: Playing nice with the MEAN stack pt2
tags:
---


So in our last post...



Essentially we will be replacing the 'database' variable, which is just an object literal, with an actual MongoDB database.

Broad steps:
- Run mongod, since we're going to be creating and accessing a database on our local machine. If you were hosting your database somewhere, you could skip this step.
- Connect to the database
- Create a schema
- Compile that schema into a model
-

Before we get too far ahead of ourselves, let's go over some Mongoose/MongoDB terminology:

<h4>Schema</h4>

The blueprint: Schemas are essentially a definition of how you want your documents to look. For example, if you want to have a document that describes a user, and you want to record specific information about each user, namely: username, first name, last name, and DoB. The schema is where you tell Mongoose the name and type (e.g. String, Boolean etc.) of information you want to store for any of your document types.
Also, if your document will need any built-in methods, you can add them to the schema definition.
Each schema automatically maps to a MongoDB collection.
{% codeblock %}
var userSchema = mongoose.schema({
  username: { type: String, required: true, unique },
  firstName: String,
  lastName: String,
  dob: Date,
})
{% endcodeblock %}


<h4>Model</h4>

The constructor: A model is a constructor that is compiled from a schema. When you want to create a new document and save it. Models give you easy access to your collections so you can find documents, or update or remove a document.
Note: Since Mongoose models are constructors, the customary approach would be to capitalize the name of the variable when declaring e.g.
{% codeblock %}
var User = mongoose.model('User', userSchema);
{% endcodeblock %}

<h4>Document</h4>

The actual info: A document is an instance of a model. Documents can be created and saved to the database, then retrieved, updated or deleted. This is the format in which you store your actual data. So if you had 100 users, then you would have 100 user documents in your database.

<h4>Collection</h4>

MongoDB documents are grouped into collections. They are kind of like tables in MySQL, but they do not enforce a schema, and thus all documents within a collection don't need to have the same fields. This is what makes NoSQL database systems like MongoDB so flexible.


Let's do this!
Step 1. Connect to the database:
In our <code>server.js</code> file, we're going to replace the mockDatabase with a real MongoDB database.
- require mongoose.
Let's require mongoose, set the address of the database and open a pending connection. I just called my database 'messages', and because I'm running it on my own machine, the address is <code>mongodb://localhost/messages</code>

{% codeblock %}
var express = require('express');
var bodyParser = require('body-parser');
var morgan = require('morgan');
var mongoose = require('mongoose');

//Set address and open connection to database
mongoose.connect('mongodb://localhost/messages');
{% endcodeblock %}

Next up, we'll store that pending connection in a variable called <code>db</code> and then check whether the connection was successful or not:

{% codeblock %}
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function() {
  console.log('database connected');
});
{% endcodeblock %}

Note that if the database doesn't exist, Mongo will automatically create it when you try to insert a collection.

Step 2. Create a schema
Next up, we need to create a Mongoose schema for our message documents (remember, we're building a simple message board). We'll only need one 'text' field for our messages, and that should be of type String.

{% codeblock %}
var messageSchema = new mongoose.Schema({
  text: String
});
{% endcodeblock %}

Note that some resources leave out the <code>new</code> keyword when calling mongoose.Schema(). You can use it or leave it out, but Mongoose will create a new instance either way. (See http://stackoverflow.com/questions/22596806/mongoose-schema-new-keyword)

Step 3. Compile our schema into a model

















