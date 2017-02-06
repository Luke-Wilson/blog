---
title: How to create, update, and remove D3 nodes in your DOM
date: 2016-09-30 06:54:09
tags:
- d3
- general-update-pattern
- data
---

![Man, these ants are getting fat...](/images/d3_gup.gif)

A few days ago I needed to create some d3 circle nodes (each joined to a datum in a data set) using D3, and then move those nodes every second to a new, random position. While this might not be the most common use of D3, the stuff I learned about data joins and D3's 'general update pattern' should serve me well the next time I get to use D3.
<!-- more -->
So... the task at hand:

We want to create a circle for each datum in our data set. We'll do this by creating a data join (see Mike Bostock's <a href="https://bost.ocks.org/mike/join/">excellent blog post</a> that goes into more detail about data joins). Essentially, we want to join one node to one datum.

The general update pattern allows you to compare your dataset to your DOM nodes and find out:
a) which data points are already joined to nodes  -- .update()
b) which data points are not yet joined to nodes  -- .enter()
c) which nodes are no longer joined to data points -- .exit()

<code>.update()</code>, <code>.enter()</code>, and <code>.exit()</code> allow you to create sub-selections of a larger selection, and then perform operations on that sub-selection.

Note: You'll need an SVG element for the below code, since that's what the base selection is looking for.

<h3>Getting your data points into your DOM using .enter()</h3>

You will start without any nodes joined to your data. How do you create a circle for each datum? Using <code>.enter()</code>.

Since the general update pattern allows us to compare our data with our nodes, we can use <code>.enter()</code> to find the data points that aren't yet joined to a DOM node, and then create a DOM node for each!

{% codeblock Creating DOM nodes using .enter() %}
//Data set
var myData = [200,310,300,350,100,250, 125, 150, 175];

//Random location generators
var randomH = function() { return (window.innerHeight - 100) * Math.random(); };
var randomW = function() { return (window.innerWidth - 250) * Math.random(); };

//Enter section: Creates nodes without prev data, adds data, adds them to svg
d3.select('svg').selectAll('circle')   // ---> Don't worry that there are no circles in SVG yet.
  .data(myData, function(d) {return d;})
  .enter()   // ---> creates subselection of data that don't yet have nodes
  .append('circle')    //---> creates circle nodes for each datum
  .attr('cx', randomW)    //---> sets x position of circle
  .attr('cy', randomH)    //---> sets y position of circle
  .attr('r', function(d) { return 5;} )  //---> sets radius of circle
  .attr('id', function(d) { return d;} )
{% endcodeblock %}

The above code selects the SVG, looks for any circles to update (there are none), then when it sees <code>.enter()</code>, it says, "Oh, OK. I need to append a circle to SVG for every data point that doesn't yet have a DOM node attached to it."

<h3>Updating your data-joined DOM nodes using .update()</h3>
Now that we have some nodes in the DOM that are joined to data, we need to select them so that we can update their X and Y coordinates. We do this using .update(). Actually... we don't even need to write <code>.update()</code>, because <code>.update()</code> is returned by the <code><i>selection</i>.data()</code>.

So, here's how we select existing data-joined nodes and update them:

{% codeblock Updating existing DOM nodes using .update() ( but really using .data() ) %}
d3.select('svg').selectAll('circle')
  .data(myData, function(d) {return d;})   //---> this returns .update() so we don't have to write it
  .transition().duration(1000)
  .attr('cx', randomW)    //---> updates x position of circle
  .attr('cy', randomH)    //---> updates y position of circle
{% endcodeblock %}

Easy, right?

<h3>Removing DOM nodes using .exit()</h3>
Ok, so now we know <code>.update()</code> creates a sub-selection of nodes that ARE joined to data, and <code>.enter()</code> creates a sub-selection of datum that AREN'T joined to nodes. Let's look at <code>.exit()</code>

<code>.exit()</code> creates a sub-selection of remaining nodes that are no longer bound to data. This might happen if you were dynamically updating the data set (maybe if the user was filtering data?) Usually, we don't want any loose nodes just floating around, so we'll probably want to remove them from our graph. So...

{% codeblock Removing left over nodes using .exit() %}
d3.select('svg').selectAll('circle')
  .data(myData, function(d) {return d;})
  .exit()
  .remove()
{% endcodeblock %}

And that's it! Now you know how to take a dataset, and either:
a) create a node for each datum that doesn't yet have a node
b) update any nodes that have been joined to data
c) remove any nodes that no longer have a join to data.

<h5>More resources:</h5>
<ul><li>Mike Bostock's <a href="https://bost.ocks.org/mike/join/">data joins</a> post</li><li>Mike Bostock's <a href="https://bl.ocks.org/mbostock/3808218">general update pattern example</a></li></ul>
