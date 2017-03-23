---
title: Playing nice with the MEAN stack - part 1
tags:
  - Express
  - Angular
date: 2017-03-21 22:04:56
---

![MEAN - preferred by cats everywhere](/images/2017/03/cat-pushes-cat-into-pool.gif)

Whether you're new to Angular, or it's been a while since you last worked in the MEAN stack, this post should give you a clear idea of how to get the basics of an Angular app up and running.
<!-- more -->
Angular is a great MVW (MV-Whatever) framework that has very useful two-way data binding between the view and the model. It also has easy dependency injection, allowing for highly modularized code without a lot of hassle. You can get an Angular app up and running really quickly, which makes it great for quickly building prototypes.

To demo some of this, we'll run through how to build a simple message board app using the MEAN stack.  In this post, we'll:
- create a basic server with a mock database (we'll add MongoDB in pt 2)
- create a front end using Angular
- hook up some comms between the front end and the back end

Our initial file structure will look like this:

{% codeblock %}
- client
  - index.html
  - index.js
- server
  - server.js
{% endcodeblock %}

<h3>Setting up the server</h3>

First things first. Let's create our basic server using Express.js.

{% codeblock server/server.js %}
var express = require('express');
var morgan = require('morgan');
var bodyParser = require('body-parser');

//create express object
var app = express();

//add useful middleware
app.use(morgan('dev'));
app.use(bodyParser.json());

//set route for static files
app.use(express.static('../client'));

//simple array as a stand-in for database
var mockDatabase = [
  {text: "hi, this is the first message"},
  {text: "hi, this is the second message"},
];

//set port and listen
var port = 3003;
app.listen(port, _ => {
  console.log('server is listening on', port);
});
{% endcodeblock %}

<h3>Setting up Angular</h3>

In our <code>index.html</code>, we'll just add some static content, load the angular CDN in the HEAD section, and just before the closing body tag, we'll load our index.js file.

In our <code>index.js</code> file, we can set up a super basic Angular app using <code>angular.module</code>, and one controller constructor:
{% codeblock SUPER basic Angular hookup. It works, but... %}
var myApp = angular.module('messageBoard', [])
myApp.controller('messageController', function($scope) {
  $scope.test = 'OMG Angular is working';
});
{% endcodeblock %}

With an app this small, having these two items in the same file is manageable, but when our app gets bigger and as we add more modules, controllers and services, it will become very messy. So let's modularize it now by creating a new module with a controller into a separate file, and then we can just inject that module into our main app module.

So let's make the following changes to our file structure:
{% codeblock %}
- client
  - messages               <---
    - messageController.js <---
  - services               <---
    - messageService.js    <---
  - index.html
  - index.js
- server
  - server.js
{% endcodeblock %}

In <code>messageController.js</code>, let's create a new module called <code>messageBoard.messageController</code> (you can shorten this if you want, I'm being deliberately verbose), and create a controller constructor called 'messageController':

{% codeblock client/messages/messageController.js%}
angular.module('messageBoard.messageController', [])
.controller('messageController', function ($scope) {
  $scope.test = 'OMG Angular is working!';
});
{% endcodeblock %}

In our <code>index.js</code>, we'll inject this module into our main module:

{% codeblock client/index.js %}
angular.module('messageBoard', [
  'messageBoard.messageController'
  ]);
{% endcodeblock %}

Now, since we're adding a new module to our app, we'll also need to load it in our index.html. And since index.js depends on the existence of messageBoard.messageController, we'll have to load the latter first. At this stage, our <code>index.html</code> should look like this:
{% codeblock client/index.html %}
<!DOCTYPE html>
<html>
  <head>
    <title>Message Board</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.6.1/angular.js"></script>
  </head>
  <body ng-app="messageBoard">
    <h1>Welcome to the message board!</h1>
    <h3>Messages:</h3>
    <div ng-controller="messageController">
      {% raw %}{{test}}{% endraw %}
    </div>
    <script src="./index.js"></script>
  </body>
</html>
{% endcodeblock %}

<h4>REFACTOR TIME!!</h4>

Following <a href="https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md">John Papa's awesome style guide for Angular 1</a>, we're going to change our controller to use controllerAs, removing the need to reference <code>$scope</code>. This is better practice because it will make us access the controller instance's properties via the dot method (e.g. <code>messages.username</code> instead of just <code>username</code>). This is easier to read, and greatly reduces the chances of collisions or mixups with other variables. So in our <code>index.html</code>, we'll change the following:

{% codeblock client/index.html %}
...
<div ng-controller="messagesController as messages">
  {% raw %}{{messages.test}}{% endraw %}
</div>
...
{% endcodeblock %}


<h3>Setting up communications</h3>

Now that we have basic functionality on both the back end and front end, we can set up some communication between the two. Specifically, we'll need to do two things:
1. Get all messages from our database
2. Create a new message

We want our controller just handling the view logic and presenting it the data it needs, not dealing with communications to the backend. For that, we can use a factory. In our services folder, open the <code>messageService.js</code> file. There we can create a new module called <code>messageBoard.messageService</code>, and add a factory to it called 'Messages'. Factories are instances (as opposed to services, which are constructors that are instantiated using the <code>new</code> keyword), and so our factory will want to return an object literal, with key-value pairs containing the names and definitions of the functions we'll use to talk to the back end about messages. Our Messages factory will have two methods: getMessages and postMessage.

{% codeblock client/services/messageService.js %}
angular.module('messageBoard.messageService', [])
.factory('Messages', ($http) => {
  return {
    getMessages: () => {
      return $http({
        method: 'GET',
        url: '/api/messages'
      })
      .then(resp => {
        console.log(resp.data);
        return resp.data;
      })
      .catch(err => console.log(err));
    },

    postMessage: (messageText) => {
      return $http({
        method: 'POST',
        url: '/api/messages',
        data: {text: messageText}
      })
      .then(resp => {
        return resp;
      })
      .catch(err => console.log(err));
    }
  };
});
{% endcodeblock %}

getMessages uses Angular's built-in $http service to send a GET request to the server at <code>/api/messages</code>, and then wait and return the response. postMessage POSTs a message to the server, waits and then returns the response. All the other logic will be sorted out on the server in server.js...

{% codeblock server/server.js %}
...
app.get('/api/messages', (req, res) => {
  res.send(mockDatabase);
});

app.post('/api/messages', (req, res) => {
  mockDatabase.push({text: req.body.text});
  res.send(`successfully posted: ${req.body.text}`);
});
...
{% endcodeblock %}


So now that we have comms set up between our message service and our server, all we have left to do is inject our service into our message controller so that we can have access to the service's methods and the data that they'll get from the server.

In our <code>messageController.js</code> file, we'll need to inject the name of our factory, 'Messages', as an argument to the controller constructor function. We'll also create a couple of methods on the controller, which I've deliberately named differently so it's a little easier to follow the chain of events:

{% codeblock messages/messageController.js %}
angular.module('messageBoard.messageController', [])
.controller('messageController', function (Messages) {
  var vm = this;
  vm.store = [];

  vm.askForMessages = () => {
    Messages.getMessages()
    .then(resp => {
      vm.store = resp;
    });
  };

  vm.sendMessage = (messageText) => {
    Messages.postMessage(messageText)
    .then(resp => {
      console.log(resp);
      vm.askForMessages();
    });
  };
};
{% endcodeblock %}


Finally, in our <code>index.html</code>, we have a couple of changes to make to our <code>ng-controller</code> div. We'll add a <code>ng-click</code> listener that will trigger the askForMessages method. We'll also add a form that, when submitted, will trigger the sendMessage method on our controller, and we'll use ng-repeat to iterate through the <code>messages.store</code> and create a new <code>li</code> element for each item.
{% codeblock client/index.html %}
...
<div ng-controller="messageController as messages">
  <div>
    <button ng-click="messages.getMessages()">Get Messages</button>
  </div>
  <form ng-submit="messages.sendMessage(purpleUnicorn)">
    <input type="text" ng-model="purpleUnicorn" placeholder="enter your message text">
    <input type="submit" />
  </form>
  <ul>
    <li ng-repeat="each in messages.store">{% raw %}{{each.text}}{% endraw %}</li>
  </ul>
</div>
...
{% endcodeblock %}

Everything should now be hooked up and working!

![](/images/2017/03/messageBoard.gif)

<h3>Hold up, wait a minute...</h3>

There's only one thing wrong. We don't have persistent data: whenever the server restarts, we lose all our messages. In the next post, we'll hook up a persistent data store using MongoDB and Mongoose.


