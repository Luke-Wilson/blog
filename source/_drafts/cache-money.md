---
title: cache money
tags:
- Redis
- caching
---

Recently I was building a small resource calendar-style app that spent a lot of time talking to an external API. Even though I'm a pretty patient guy, I started getting a bit tired with waiting for the API calls to be returned, particularly on load. I started thinking "Oh no! This sluggish API is going to make people think that MY app is slow, and they're going to blame ME!". Clearly, something had to be done.

What I proposed was to set up a Redis cache to remember the common calls I was making to the API. When users booted up the app, it would load to today's date and request all the room bookings for today's date. This meant that, especially for the first API call, that the data the API would be returning would be the same for all users.

Cold, hard c-c-cache!

---------------------------------------------
- What is Redis?
---------------------------------------------
Let's quickly explore what is Redis, and why it's so good at this sort of thing.

Redis is a NoSQL key-value data store. It is in-memory storage. This means that while on the one hand, you probably won't have terabytes of storage because you are limited by the amount of memory on the machine where Redis is running, BUT on the other hand, it is crazy fast. Crazy. It's like the Flash Gordon of caches... Cache Gordon, if you will.

---------------------------------------------
- 5 Different Types of Data Structures
---------------------------------------------

Another cool thing about Redis is that it's an advanced key-value store, meaning your values don't have to be strings. They CAN be strings, but they also can be sets, sorted sets, hashes and lists! And Redis lets you perform operations on these values allowing you to do stuff like (FIND COOL EXAMPLE).

--values don't have to be strings. Explain Redis 5 types of data structure
  - strings
      Redis can store strings up to a maximum storage size of 512MB. That's A LOT of characters.

  - sets
      Collections of unique strings
      Adding, removing and testing for existence are all O(1) - nice and fast!

  - sorted sets
      Collections of unique strings that each have their own 'score' which is used to order them.
      With sorted sets you can add, remove, or update elements in a very fast way: O(log(n)), so not as fast as the unordered set.
      You might use a sorted set for a leaderboard for a sport competition, e.g. a competition between bears to see who could scare the most campers. And they come with some useful methods. Just use ZREVRANGE to sort the bears by score from highest to lowest, and you could use ZRANK to get their ranking on the leaderboard.

  - hashes
      Great for storing objects, say a user object with fields like name, surname, age, address, least favorite bear, etc.
      Interestingly, Facebook saves user configs as hashes (LINK: https://www.slideshare.net/RedisLabs/using-redis-at-facebook).

  - lists
      Lists of strings sorted by insertion order (either left or right push - LPUSH / RPUSH)
      Adding to list (left or right) is O(1), but accessing elements within the list is O(N)
      Lots of useful commands: LPUSH, LRANGE, LTRIM
      You could use a list to store the latest posts for a blog. Twitter uses a type of list for its Home Timelime and User Timeline (link: http://highscalability.com/blog/2014/9/8/how-twitter-uses-redis-to-scale-105tb-ram-39mm-qps-10000-ins.html). 


---------------------------------------------
- Why cache?
---------------------------------------------
One of the biggest reasons to cache data is to reduce the need for time consuming and costly database or external API calls, especially if that API is sloooow.

If the data itself doesn't change super often, then it might be a good candidate for caching. 

You do want to be careful though, as you always want to be displaying the up-to-date data. This means you need to account for any user interaction or anything else that might cause the data to change. When you see info that you know is out of date, there's nothing worse. Well, maybe except for bears. 


---------------------------------------------
How to!
---------------------------------------------



---------------------------------------------
- How to (with Node):
  - install Redis: I recommend using Homebrew: 
  `brew install redis`
https://redis.io/topics/quickstart
  - Add redis NPM package (npm install redis)
  - require redis
  - create a new redis client
    - Add an on connect handler to verify
  - Storing strings
    - client.set(key, value)
  - Getting values:
    - client.get(key, errback);
  
  - Adjust route by adding your getBeersCache function between the route and the API call function: 
  e.g. app.get('/beers', getBeersCache, getBeersAPI);

  - Make sure you pass (req, res, next) to your getBeersCache function, so that if you get nothing back from your cache call, you can just call next() and it will jump to the next function in the route.

  - Since Brewery DB is giving us back a JSON object, let's store it as an object using Redis' hash.
  





  - Storing hashes
    - client.hmset(hashKey, k1, v1, k2, v2, k3, v3...);
    or
    - client.hmset(hashKey, {k1: v1, k2: v2, ...});
  - Getting hashes
    - client.hgetall(hashKey, errback)

  




Pub/Sub - (LINK: https://redis.io/topics/pubsub) - The sender of a message (publisher) doesn't send directly to the recipient (subscriber). Instead, the publisher publishes their message to a stream, and the subscriber *subscribes* to that stream.


launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist