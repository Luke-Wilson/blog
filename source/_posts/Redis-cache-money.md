---
title: 'Redis: Cache money'
tags:
  - Redis
  - caching
date: 2017-08-13 23:18:54
---


![Cold, hard c-c-cache!](/images/2017/08/bank-vault-tim-evans.jpg)

Recently I was building a small resource calendar-style app that spent a lot of time talking to an external API. Even though I'm a pretty patient guy, I started getting a bit tired with waiting for the API calls to be returned, particularly on load. I started thinking "Oh no! This sluggish API is going to make people think that MY app is slow, and they're going to blame ME!". So, I started thinking, maybe I could use **Redis** to cache the initial calls (which would usually be the same info for all users).

<!-- more -->

<h3>What is Redis?</h3>

Let's quickly explore what Redis is, and why it's so good at this sort of thing.

Redis is a NoSQL key-value data store. It is in-memory storage, which means it is crazy fast. Crazy. It's like the Flash Gordon of caches... Cache Gordon, if you will. The down side is that, since you probably don't have a tonne of memory just lying around, you're not going to be able to store everything in there like you might a SQL database (but we're just using it as a cache, so we won't be storing all that much in there anyway).

<h3>Why cache?</h3>

One of the biggest reasons to cache data is to reduce the need for time consuming and costly database or external API calls, especially if that API is sloooow.

If the data itself doesn't change super often, then it might be a good candidate for caching.

You do want to be careful though, as you always want to be displaying the up-to-date data. This means you need to account for any user interaction or anything else that might cause the data to change. When you're testing your site and you see info being displayed that you know is out of date, there's nothing worse. And you definitely don't want to be showing out of date info to your users.

<h3>5 Different Types of Data Structures</h3>

Another cool thing about Redis is that it's an advanced key-value store, meaning your values don't have to be strings. They CAN be strings, but they also can be sets, sorted sets, hashes and lists! And Redis lets you perform operations on these values too. Check out the {% link Redis docs https://redis.io/documentation %} for examples.

**strings**: Redis can store strings up to a maximum storage size of 512MB. That's A LOT of characters.

**sets**: Collections of unique strings. Adding, removing and testing for existence are all O(1) - nice and fast!

**sorted sets**: Collections of unique strings that each have their own 'score' which is used to order them. With sorted sets you can add, remove, or update elements in a very fast way: O(log(n)) - still not as fast as the unordered set though.

You might use a sorted set for a leaderboard for a sport competition, e.g. a competition between bears to see who could scare the most campers. And they come with some useful methods. Just use <code>ZREVRANGE</code> to sort the bears by score from highest to lowest, and you could use <code>ZRANK</code> to get their ranking on the leaderboard.

**hashes**: Great for storing objects, say a user object with fields like name, surname, age, address, least favorite bear, etc. Interestingly, {% link Facebook saves user configs as hashes  https://www.slideshare.net/RedisLabs/using-redis-at-facebook %}.

**lists**:Lists of strings sorted by insertion order (either left or right push - <code>LPUSH</code> / <code>RPUSH</code>). Adding to list (left or right) is O(1), but accessing elements within the list is O(N).

Lots of useful commands: <code>LPUSH</code>, <code>LRANGE</code>, <code>LTRIM</code>. You could use a list to store the latest posts for a blog.  {% link Twitter uses a type of list for its Home Timelime and User Timeline http://highscalability.com/blog/2014/9/8/how-twitter-uses-redis-to-scale-105tb-ram-39mm-qps-10000-ins.html %}.

<h3>A quick demo</h3>

I'll run through a super basic example where we'll cache an API response in Redis as a JSON string. Whenever a request comes through, our server will first check our cache for the info, and if it doesn't exist, the server will send a request to the external API, save the response in the cache and return the info to the client.

{% blockquote %}
NOTE: If you want to follow along, you'll need to make sure Redis is installed and you have a local redis-server running. I recommend using Homebrew: <code>brew install redis</code>

See also https://redis.io/topics/quickstart
{% endblockquote %}

I have a Node Express server (full code at the end of this post) that has just one GET route:  <code>"/beers/:id"</code>. Any requests to this route will call the <code>getBeersFromAPI</code> route handler function, which will then send off a request to an API to retrieve info about beers matching the glassware ID provided.

{% codeblock %}
// Route
app.get('/beers/:id', getBeersFromAPI);

// Request handler
function getBeersFromAPI (req, res) {
  request(`http://api.brewerydb.com/v2/beers?key=${API_KEY}&glasswareId=${req.params.id}`,
    function(err, response) {
      // Return the response to the client
      res.send("<h1>From API:</h1>" + JSON.stringify(response));
    }
  );
};
{% endcodeblock %}

The first thing we'll need to do is add Redis to our server and set up a Redis connection:

{% codeblock %}
var redis = require('redis');

// Create redis client and connect to it.
var client = redis.createClient();
client.on('connect', _ => console.log('redis connected'));
{% endcodeblock %}

Next, we'll create a new route handler, <code>getBeersFromCache</code>, which will check the cache for the requested info, and return that info if it exists. We're going to use Redis' {% link get https://redis.io/commands/get %} command to see if we get the info from our cache that matches the key provided (e.g. <code>"glasswareId-4"</code>).

{% codeblock %}
function getBeersFromCache (req, res, next) {
  client.get(`glasswareId-${req.params.id}`, (err, result) => {
    if (err) {
      console.log('error in getBeersFromCache', err);
      next();
      return;
    }
    if (result) {
      // Yay! The info exists in our cache. Let's return it to the client
      res.send('<h1>From REDIS:</h1>' + JSON.stringify(result));
    } else {
      // Ooops! I guess we don't have that info in our cache yet. Better call the next route handler.
      next();
    }
  });
}
{% endcodeblock %}

If we get an error, or if there is no result, we just call next() which will call the next route handler in the chain (see below). In our case, it will be <code>getBeersFromAPI</code>.

{% codeblock %}
// Route (now with extra route handler)
app.get('/beers/:id', getBeersFromCache, getBeersFromAPI);
{% endcodeblock %}

We'll also modify the <code>getBeersFromAPI</code> function to not only return the information to the client, but ALSO save that information to the cache so that next time a request is made for that info, we'll be able to just grab it from the cache. We use Redis' {% link set https://redis.io/commands/set %} command by giving it a key (e.g. 'glasswareId-4') then the value (e.g. our response as a JSON string). We will also set a time limit (60 seconds) for that key-value pair to expire. This is optional.

{% codeblock %}
function getBeersFromAPI (req, res) {
  request(`http://api.brewerydb.com/v2/beers?key=${API_KEY}&glasswareId=${req.params.id}`,
    function(err, response) {
      // Store value as a string in Redis (for next time) and return string to client
      client.set(`glasswareId-${req.params.id}`, JSON.stringify(response), 'EX', 60);
      res.send("<h1>From API:</h1>" + JSON.stringify(response));
    }
  );
};
{% endcodeblock %}

{% blockquote %}
Make sure you pass 'next' as your third parameter to your getBeersCache function, so that if you get nothing back from your cache call, you can just call next() and it will jump to the next function in the route.
{% endblockquote %}

That's pretty much it! Check out the differences in response times:

![](/images/2017/08/API_vs_Redis_times.png)

Obviously this is just a basic example, and its utility in the real world is a bit limited without further tweaks, but hopefully it gives you enough to go on. From here, you can do some fun extra exploration like manually deleting values, playing with expiration times, flushing the database, or storing values as sets or hashes. Have fun!

{% codeblock Full server code %}
var express = require('express');
var bodyParser = require('body-parser');
var morgan = require('morgan');
var path = require('path');
var request = require('request');
var API_KEY = require('./secrets.js')["API_KEY"]; //Get your own at brewerydb.com
var redis = require('redis');

// Create redis client and connect to it.
var client = redis.createClient();
client.on('connect', _ => console.log('redis connected'));

// Set up server
var app = express();
app.use(express.static(path.join(__dirname, '../client')));
app.use(bodyParser.json());
app.use(morgan('dev'));

//Route
app.get('/beers/:id', getBeersFromCache, getBeersFromAPI);

//Request handlers
function getBeersFromCache (req, res, next) {
  client.get(`glasswareId-${req.params.id}`, (err, result) => {
    if (err) {
      console.log('error in getBeersFromCache', err);
      next();
      return;
    }
    if (result) {
      // Return string
      res.send('<h1>From REDIS:</h1>' + JSON.stringify(result));
      client.del(`glasswareId-${req.params.id}`); //manually delete the key for easy testing: one API, one cache, one API, one cache...
    } else {
      next();
    }
  });
}

function getBeersFromAPI (req, res) {
  request(`http://api.brewerydb.com/v2/beers?key=${API_KEY}&glasswareId=${req.params.id}`,
    function(err, response) {
      if (err) {
        console.log(err);
        return;
      }
      // Store as string and return string
      client.set(`glasswareId-${req.params.id}`, JSON.stringify(response), 'EX', 10);
      res.send("<h1>From API:</h1>" + JSON.stringify(response));
    }
  );
};

// Listen
var port = 3000;
app.listen(port, () => {
  console.log('Listening on port', port);
});

{% endcodeblock %}
