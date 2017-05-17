---
title: Setting up React with Webpack
date: 2017-02-01 16:01:42
tags:
- React
- Webpack
- Babel
- Express
---

![Look how happy he is to have gotten Webpack up and running so easily!](/images/2017/02/coffee.jpg)

One of my amazing colleagues, Ivey Topaz, wrote a very helpful blog on <a href="http://iveytopaz.com/2016/10/25/starting_react_from_scratch/">setting up React from scratch using Grunt</a>.

And since I haven't had the chance yet to play around with Webpack, I thought it would be useful to figure out for myself how to build a React project from scratch using Webpack. Come along for the ride and make your very own barebones React + Webpack + Babel repo!
<!-- more -->
<h3>Before we start:</h3>

If you haven't got Webpack installed globally, do it meow by entering <code>npm install -g webpack</code> in your terminal.

Also, feel free to <a href="https://github.com/Luke-Wilson/react-webpack-starter">fork my repo</a> if you want to compare your code.

<h3>Broad steps:</h3>

1. Initialize a npm repo
2. Add a compiler (Babel)
3. Add Webpack and a loader
4. Configure Webpack
5. Use Express to set up a server
6. Add React and create a test component


<br /><h3>1. Initialize an npm repo and set up basic file structure</h3>

Easy! Inside our project's directory, type <code>npm init</code> and accept default settings.

Now, get the basic directory structure set up. Create the following sub-directories at the root level:
<ul><li>src</li><li>www</li><li>server</li></ul>

Inside our 'src' directory, we'll create a basic <code>app.js</code> file, with just a single log statement for now:
{% codeblock src/app.js %}
console.log('Hello from our app.js file!');
{% endcodeblock %}

Inside our 'www' directory, we'll create a basic <code>index.html</code> file, and load in <code>bundle.js</code>. Don't worry that bundle.js doesn't exist yet... Webpack will create it for us. We'll also add a <code>div</code> with the id of "app", which will eventually be the target of our React app.
{% codeblock www/index.html%}
<html>
<head>
</head>
<body>
  <h1>This is our index.html file</h1>
  <div id="app"></div>
  <script src="/bundle.js" ></script>
</body>
</html>
{% endcodeblock %}

<br /><h3>2. Add a compiler (Babel)</h3>

{% codeblock %}
npm install --save babel-core babel-preset-es2015 babel-preset-react
{% endcodeblock %}

Babel allows us to give it ES6 and JSX (which is sooo nice when working with React components), and in return it gives us browser-ready ES5! (Well mostly, anyway.)

So, without worrying about the browser freaking out, we can write beautiful, efficient code like this:

{% codeblock example.js %}
  const someComponent = () => (
    <div>Wait... HTML inside JavaScript? Oh that's right! JSX!</div>
  );
{% endcodeblock %}

Before we move on, we should create the .babelrc file to tell Babel to use the ES6 and react presets we have installed. Create <code>.babelrc</code> in the root directory.
{% codeblock .babelrc %}
{
  "presets": ["es2015", "react"]
}
{% endcodeblock %}


<br /><h3>3. Add Webpack and a loader</h3>

{% codeblock %}
npm install --save webpack babel-loader
{% endcodeblock %}

Unfortunately Babel doesn't solve all our problems though. During the course of compiling, Babel will transform any 'import' statements into 'require' statements. And of course, 'require' is back-end code and won't do us much good in the browser.

Webpack is a module bundler. With the help of babel-loader, it will take our modules and dependencies and bundle them up into a browser-friendly bundle.js file. Thanks Webpack!

{% blockquote %}
(Side note: Webpack can even generate multiple bundle files, a process known as <a href="https://webpack.github.io/docs/code-splitting.html">code splitting</a>, for when one single bundle file is not efficient e.g. for very large web apps.)
{% endblockquote %}


<br /><h3>4. Configure Webpack</h3>

Now let's create our <code>webpack.config.js</code> file, which is where we'll keep all of our configuration details. Essentially this file will define the <code>context</code> (the path to your client-side JavaScript folder where your entry point will be), the <code>entry</code> point (file or files that we want to include in our build), <code>output</code> (what Webpack will name the file it builds - in our case 'bundle.js'), <code>module.loaders</code> (how modules should be handled before being added to our bundle). Lastly, <code>resolveLoader</code> and <code>resolve</code> are so Webpack knows where to find loaders (e.g. Babel) and our npm packages respectively.

In the root diretory, create a file called <code>webpack.config.js</code> and drop the following code in there.
{% codeblock webpack.config.js %}
var path = require('path');
module.exports = {
  context: path.join(__dirname, 'src'),
  entry: [
    './app.js',
  ],
  output: {
    path: path.join(__dirname, 'www'),
    filename: 'bundle.js',
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loaders: ['babel-loader'],
      },
    ],
  },
  resolveLoader: {
    modules: [
      path.join(__dirname, 'node_modules'),
    ],
  },
  resolve: {
    modules: [
      path.join(__dirname, 'node_modules'),
    ],
  }
};

{% endcodeblock %}


That's most of the hard stuff done! Now to get our server going...

<br /><h3>5. Add Express and set up a server</h3>

{% codeblock %}
npm install --save express webpack-dev-middleware
{% endcodeblock %}

Let's just create a standard server in <code>server.js</code>. We'll add some necessary code to get Webpack running. We need to:
1. create our <code>compiler</code> object from webpack and our config
2. use our <code>webpack-dev-middleware</code> with our compiler object and input some options


{% codeblock server/server.js %}
var path = require('path');
var express = require('express');

//require Webpack, middleware and config
var webpackDevMiddleware = require('webpack-dev-middleware');
var webpack = require('webpack');
var webpackConfig = require('../webpack.config.js');

var app = express();

//create compiler object using webpackConfig
var compiler = webpack(webpackConfig);

// set path for static files
app.use(express.static(path.join(__dirname, '../www')));

//use webpack middleware with our compiler object and some options
app.use(webpackDevMiddleware(compiler, {
  hot: true,
  filename: 'bundle.js',
  publicPath: '/',
  stats: {
    colors: true,
  },
  historyApiFallback: true,
}));

var port = 3003;

app.listen(port, () => {
  console.log(`server listening at ${port}`)
});
{% endcodeblock %}


Now that's all set up, we can test everything to make sure it's working. In your terminal, at your project's root directory, type <code>node server/server.js</code>. Webpack should tell you that it 'compiled successfully', your server should be running, and if you navigate to http://localhost:3003 in your browser, you should see the log output from <code>app.js</code> in the browser's console.


<br /><h3>6. Add React and create a test component</h3>

Sweet! So if everything's working, now we just need to add React.

{% codeblock %}
npm install --save react react-dom
{% endcodeblock %}

Let's create a basic component that just adds a exclamation point to the text in a button. In your 'src' directory, create a file called <code>StatefulComponent.js</code> and add the below code.

{% codeblock StatefulComponent.js %}
import React from 'react';

class StatefulComponent extends React.Component {
  constructor() {
    super();
    this.state = {
      buttonText: "Click me and see what happens"
    };
  }

  addExclamation () {
    this.setState({
      buttonText: this.state.buttonText + "!"
    });
  }

  render() {
    return (
      <button onClick={()=> {
        this.addExclamation();
      }}>
      {this.state.buttonText}
      </button>
    )
  }
}

export default React.createElement(StatefulComponent);
{% endcodeblock %}

Lastly, let's import our new component into <code>app.js</code> and render it to the DOM.

{% codeblock app.js %}
import React from 'react';
import ReactDOM from 'react-dom';

import StatefulComponent from './StatefulComponent';

ReactDOM.render(StatefulComponent, document.getElementById('app'));

{% endcodeblock %}


<br /><h3>Try it out!</h3>
So, hopefully when you start up your server, Webpack should compile successfully and when you view your app in the browser, you should see the button with the very exciting functionality of adding an exclamation point each time you click it!

Think of all the things you can do with this button!! Or just build something cool in React knowing that Webpack is up and running.






