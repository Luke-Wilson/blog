---
title: Intro to Flexbox - Part 1
date: 2017-02-06 16:12:11
tags:
- CSS
- Flexbox
---

![Flexbox is so awesome, it might just make vanilla your new favorite flavor.](/images/2017/02/ice-cream.jpg)

For a long time, I hated vanilla CSS. I just didn't get it. Why won't my div, image, input field, or whatever go where I want it to go?! How do I center that thing? NO, NOT THERE DAMMIT!

And then I tried out CSS3's Flexbox and then my life was filled with joy!

So without further ado, here is part 1 of my **Intro to Flexbox** series of posts.

<!-- more -->
{% blockquote %}
<a href="https://jsfiddle.net/lukewilson/qhshu9hn/5/">Here is a fiddle I prepared earlier</a> so you can try stuff out as you read along.
{% endblockquote %}

<br /><h3>So, what is Flexbox?</h3>

Flexbox, short for 'flexible boxes', was introduced with CSS3. It gives us an easy-to-use box model for laying out user interfaces. It is great for aligning items and distributing space among those items within their parent container. It adapts well when the dimensions of items or the container change, intelligently filling the available space.

It is also 'direction-agnostic' meaning you can flip the layout on its side, or reverse it, without changing the orientation of the content itself. Missy Elliot would be stoked.

{% img [] http://s2.quickmeme.com/img/d6/d63c623afc63f60d7de499dac284a29e5c75b42ef8bc6a163e016733e9da03e5.jpg 200 [] [title text []] %}

<h3>Flexbox: The basics.</h3>

<h4><strong>Flex containers and flex items</strong></h4>

To get started with Flexbox, you'll need to create a flex container. This is as adding <code>display: flex;</code> to an element's CSS.

{% codeblock %}
.container {
  display: flex;
}
{% endcodeblock %}

All children of a flex container automatically become 'flex items', which means you can do 'flexy' stuff with them.

<br /><h4><strong>Axes and flex-direction</strong></h4>

Before we go any further, we should talk about axes. And not the wood-chopping variety.

A flex container has a main axis and a cross axis. With default settings, it should look like this:
![Image: css-tricks.com](https://cdn.css-tricks.com/wp-content/uploads/2011/08/flexbox.png)

Now, while the main axis by default goes from left to right, that may not always be the case because Flexbox is direction-agnostic.

Using <code>flex-direction</code> in your flex container will change the container's orientation and thus change your axes around. For instance, <code>flex-direction: row-reverse;</code> will make your main axis go from right to left, and <code>flex-direction: column</code> will change your main axis to go from top to bottom, and your cross axis would then be horizontal. (The content inside your flex items won't be on it's side though, just the orientation of the container.)

{% blockquote %}
<code>flex-direction</code> summary:
  - Add this property to your flex container
  - Use it to change the container's orientation (main axis and cross axis)
  - Some useful values:
    - row (default)
    - row-reverse
    - column
    - column-reverse
  - more info: http://www.w3schools.com/cssref/css3_pr_flex-direction.asp
{% endblockquote %}

<br /><h3>Aligning, justifying and centering</h3>

Even after being shown a zillion times, trying to place something where you want it within a div is always a royal pain in the butt.

But not so with Flexbox! Using a combination of <code>justify-content</code> and <code>align-items</code> in your flex container, it's easy to distribute and align items where you want them.

**justify-content** is how your items are aligned along the *main* axis (by default left to right, but remember we can change this using flex-direction).

**align-items** does essentially the same thing as <code>justify-content</code> but along the *cross* axis. Interestingly, the default value is 'stretch', which means the items will stretch along the cross axis of the container.

![](/images/2017/02/jc-vs-ai2.png)


{% blockquote %}
<code>justify-content</code> and <code>align-items</code> summary:
  - Add these properties to your flex container
  - Use <code>justify-content</code> to position/distribute your flex items along the *main* axis. Useful values:
    - flex-start (default)
    - flex-end
    - center
    - space-around
    - space-between
  - Use <code>align-items</code> to position/distribute your flex items along the *cross* axis. Useful values:
    - stretch (default)
    - flex-start
    - center
    - flex-end
    - baseline (useful for aligning the baselines of different-sized text across multiple items)
  - more info:
    - justify-content: http://www.w3schools.com/cssref/css3_pr_justify-content.asp
    - align-items: http://www.w3schools.com/cssref/css3_pr_align-items.asp
{% endblockquote %}

HINT: If things are going a little awry when positioning, a good debugging question to ask is: "Where is my main axis, and where is my cross axis?"

<br /><h3>Ordering items</h3>

By default, all flex items have an <code>order</code> value of 0. However, if you wanted to reorder the items, you can set a flex item's order value manually. Then, Flexbox will rearrange your items by their order values, from least to greatest.

For example, if you wanted a particular item to be the first item, you could set it's order to -1, like I've done with the item with class 'd' below:

{% codeblock %}
.d {
  order: -1;
}
{% endcodeblock %}

![](/images/2017/02/dabc.png)

<br /><h3>Part 2 coming soon...</h3>
We'll look at wrapping and <code>align-content</code>, as well as <code>flex</code> (which includes <code>flex-grow</code>, <code>flex-shrink</code> and <code>flex-basis</code>).

In the meantime, here are some great resources for you to try out:
- Wes Bos' AWESOME (and free) Flexbox course, <a href="https://flexbox.io/">What the Flexbox?</a>
- <a href="flexboxfroggy.com">FlexboxFroggy</a>, a fun game to practice the basics of aligning and positioning content
- <a href="https://css-tricks.com/snippets/css/a-guide-to-flexbox/">A Complete Guide to Flexbox</a>, from css-tricks.com

