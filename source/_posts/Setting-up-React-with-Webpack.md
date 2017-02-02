---
title: Setting up React with Webpack
date: 2017-02-01 16:01:42
tags:
- React
- Webpack
- Babel
---
![Look how happy he is to have gotten Webpack up and running so easily!](/images/coffee.jpg)

One of my amazing colleagues, Ivey Topaz, wrote a very helpful blog on <a href="http://iveytopaz.com/2016/10/25/starting_react_from_scratch/">setting up React from scratch using Grunt</a>.

Having heard a lot about Webpack, and not having a tonne of experience building React projects, I thought it would be useful to figure out how to build a React project from scratch using Webpack. So... here we go!


<h4>Before we start:</h4>If you haven't got webpack installed globally, do it meow.
<code>npm install -g webpack</code>


<h4>Broad steps:</h4>1. Initialize a npm repo
2. Add a compiler (Babel)
3. Add and configure Webpack
4. Use Express to set up a server
5. Add React and create a test component


<h4>1. Initialize an npm repo and set up basic file structure</h4>Easy! Inside our project's directory, type:

<code>npm init</code>

Now, get the basic directory structure set up. Create the following sub-directories at the root level:
<ul><li>src</li><li>www</li><li>server</li></ul>


<h4>2. Add a compiler (Babel)</h4>

<code>npm install --save babel-core babel-preset-2015 babel-preset-react</code>

Babel allows us to give it ES6 and JSX (which is sooo nice when working with React components), and in return it gives us browser-ready ES5! (Well mostly, anyway.)

So, without worrying about the browser, we can feed it stuff like the efficient code below. Thanks Babel!
{% codeblock %}
  const someComponent = () => (
    <div>Wait... HTML inside JavaScript? Oh that's right! JSX!</div>
  );
{% endcodeblock %}
Before we move on, we should create the .babelrc file to tell Babel to use the presets we have installed.
{% codeblock %}
{
  "presets": ["es2015", "react"]
}
{% endcodeblock %}


<h4>3. Add and configure Webpack</h4>
<code>npm install --save webpack babel-loader</code>

Unfortunately Babel doesn't solve all our problems though. During the course of compiling, Babel will transform any 'import' statements into 'require' statements. And of course, 'require' is back-end code and won't do us much good in the browser.

Webpack is a module bundler. With the help of babel-loader, it will take our modules and dependencies and bundle them up into a browser-friendly bundle.js file. Thanks Webpack!

NOTE: Webpack can even generate multiple bundle files, a process known as <a href="https://webpack.github.io/docs/code-splitting.html">code splitting</a>, for when one single bundle file is not efficient (e.g. for very large web apps).

--Configure Webpack
Now let's create our webpack.config.js file, which is where we'll keep all of our configuration details. In the interest of brevity, I'm going to skip over some of the details here, but essentially this file will define the entry point (file or files that we want to include in our build), output (what Webpack will name the file it builds - in our case 'bundle.js'), module.loaders (how modules should be handled before being added to our bundle). resolveLoader and resolve are so Webpack knows where to find loaders (e.g. Babel) and npm packages respectively.

ADD CODE FOR webpack.config.js

That's the hard part done!

<h4>4. Add Express and set up a server</h4>

<code>npm install --save express webpack-dev-middleware</code>

Let's just create a standard server in server.js. And we'll add some necessary code to get webpack running:

a) webpack-dev-middleware

b) compiler object

{% codeblock %}
var path = require('path');
var express = require('express');

//require webpack, middleware and config
var webpackDevMiddleware = require('webpack-dev-middleware');
var webpack = require('webpack');
var webpackConfig = require('../webpack.config.js');

var app = express();

//create compiler object using webpackConfig
var compiler = webpack(webpackConfig);

// set path for static files
app.use(express.static(path.join(__dirname, '../www')));

//use webpack middleware with our compiler object and options
app.use(webpackDevMiddleware(compiler, {
  hot: true, //hot reloading?
  filename: 'bundle.js',
  publicPath: '/',
  stats: {
    colors: true,
  },
  historyApiFallback: true,
}));

app.listen(port, () => {
  console.log(`server listening at ${port}`)
});
{% endcodeblock %}


Finally, let's test everything is working:

  node server/server.js




<h4>5. Add React and create a test component</h4>



