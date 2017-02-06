---
title: Create a simple server using Node.js (without using Express)
date: 2016-10-04 20:49:51
tags:
- node-js
- nodemon
- server
- terminal
---
![](/images/2016/nodejs.png)

Here is a really easy way to create your own server using Node.js. Usually, you would use Express with Node to build a server, but sometimes its instructive to make life a little harder.
<!-- more -->
Before we go any further, make sure you have Node installed. There are some good instructions in <a href="http://blog.npmjs.org/post/85484771375/how-to-install-npm">this post from NPM</a>.

So, here are the broad steps:
1. Create a new javascript file and open it in your favourite text editor.
2. Write some code and create your server.
3. Start your server using <a href="https://github.com/remy/nodemon">nodemon</a> (Node monitor).
4. Test your server.

<h3>Step 1: Create a file called myServer.js using the terminal</h3>This bit is easy, but for those of you not yet familiar with working in the terminal, you can use <code>mkdir</code> to make a directory, <code>cd</code> to change directories, and <code>touch</code> to create a file.

![Green on black... sooo Matrix](/images/ns1.png)


<h3>Step 2: Create your server</h3>This bit is also easy, but a little bit more nuanced than step 1. First we need to make Node's http module available for use. We do this by using <code>require</code>. We also can start sketching out the rest of the major parts of code i.e. creating the server, and making it listen.
![The barebones of our server code](/images/ns3.png)

We'll store our server object in a variable, aptly named 'server'. http's <code>createServer</code> method takes a callback function, which takes a request argument and a response argument. The request argument is what the server receives from the client when it makes a server request. The response is what you need to build and then send back to the client using <code>response.end()</code>. (If you don't have a call to <code>response.end()</code> the request will timeout for the client.)
![Telling the server what to do when it receives a request](/images/ns4.png)
Above, we've just added a <code>console.log</code>, which we will see in the server log. We've also put a string inside of our <code>response.end()</code>, which is what will get returned. Obviously, this is just a placeholder, as a server that just returns the same string over and over would not be very useful.

Now, the last bit of our code will be to tell the server to listen to a given port. To do this, we use <code>listen</code>, to which we'll pass a port number (I'm using 3002) and a callback. The callback is run when the port starts listening, so I've just put another <code>console.log</code> so I can see in the terminal that the server is listening.

![Tell the server to listen on a given port](/images/ns5.png)

![Our finished code!](/images/ns6.png)


<h3>Step 3: Start your server</h3>Now that our server code is finished, we can start our server using the terminal. <a href="https://github.com/remy/nodemon">Nodemon</a> is great for this because it monitors for changes to your server file, however you can also just start your server using <code> node myServer.js</code>. (Make sure you run this command from within the same directory as your server file.)

![Start your server](/images/ns9.png)


<h3>Step 4: Test your server</h3>To test your server, just open up your browser and type localhost:3002 in the URL bar. You should see the response message in the browser window, and you should also see that you've received a request by checking the server log (in the terminal).

![Start your server](/images/ns12.png)

That's it! Now you can quickly and easily set up a basic server using Node.js. From there, you can continue by writing out some code to help you deal with and take action on different types of requests (e.g. GET, POST, OPTIONS, DELETE, etc.)

<h5>More resources:</h5>
<ul><li><a href="https://github.com/remy/nodemon">nodemon</a></li><li><a href="https://nodejs.org/api/http.html">Node.js http documentation</a></li></ul>