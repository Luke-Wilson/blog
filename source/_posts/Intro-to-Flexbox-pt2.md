---
title: Intro to Flexbox - Part 2
tags:
  - CSS
  - Flexbox
date: 2017-02-09 18:17:48
---

![Flexbox. This cat knows what's up.](/images/2017/02/cat_in_box.jpg)

This is part two of my Intro to Flexbox. In [part one](/2017/02/06/Intro-to-Flexbox/) we looked at Flexbox basics including <code>display: flex;</code>, <code>flex-direction</code>, <code>justify-content</code>, and <code>align-items</code>.

In this post, we'll play around with resizing flex items based on available space, wrapping them to new lines, and changing how those lines are distributed throughout the parent container.

<!-- more -->
{% blockquote %}
Here is a <a href="https://jsfiddle.net/lukewilson/qhshu9hn/10/">JSFiddle that I prepared earlier</a> for you to play with as you read this post.
{% endblockquote %}

<h3>flex... or flex-grow, flex-shrink, and flex-basis</h3>

One of the best things about Flexbox is that if you have too much extra space, or not enough space, your layout won't break. It will intelligently figure out what size things should be and where they should go.

The <code>flex</code> property does three things for a flex item. It tells the item:
- how it should scale itself up when there is too much space
- how it should scale down when there is not enough space
- what its initial size should be.

Let's unpack <code>flex</code>:

{% codeblock %}
.item {
  flex: 1 5 400px;
}
{% endcodeblock %}
This is actually shorthand for:
{% codeblock %}
.item {
  flex-grow: 1;
  flex-shrink: 5;
  flex-basis: 400px;
}
{% endcodeblock %}

<code>flex</code> is a combination of <code>flex-grow</code>, <code>flex-shrink</code> and <code>flex-basis</code>, so let's look at those properties individually first.

<br /><h4><strong>flex-basis</strong></h4>

![](/images/2017/02/fb50120.png)

<code>flex-basis</code> lets you set the initial size of a flex item before any additional space is added to it via <code>flex-grow</code> (or any space is removed from it via <code>flex-shrink</code>). It can either be an absolute value, or a percentage based on its container's width (or more specifically, the container's length along the main axis).

In the above examples, there is some white space left over within the parent container. This is where Flexbox excels. Whenever there is too much space, we can give our items a <code>flex-grow</code> value to resize them using the extra space.

<br /><h4><strong>flex-grow</strong></h4>

![](/images/2017/02/fg.png)

When we have extra space, should we give some to each item? If so, do we give them equal amounts, or should one item get more than the others?

The value for <code>flex-grow</code> represents the growth factor for any item. So when we give all our flex items a property of <code>flex-grow</code> with the same value (e.g. 1), we're saying, "*when there is extra space on that row*, that space should be divvied up evenly for all flex items."

If however, we gave one item a <code>flex-grow</code> value of <code>3</code>, then *when there is extra space on that row* that item would receive three times as much extra space as any item with a <code>flex-grow</code> value of <code>1</code>.

An item with the default <code>flex-grow</code> value (<code>0</code>) will just stay at its <code>flex-basis</code> size when there is extra space... it won't grow (as seen earlier in the two <code>flex-basis</code> examples).

<br /><h4><strong>flex-shrink</strong></h4>

![](/images/2017/02/fs.png)

When we don't have enough space, how do we shrink an item down? (i.e. Will it shrink if the flex items on that line overflow the flex container, and if so, by what factor?). If we set an item's <code>flex-shrink</code> to <code>2</code>, then it will shrink twice as much as an item with <code>flex-shrink: 1;</code> (which, incidentally, is the default value).

An item set to <code>flex-shrink: 0;</code> will not shrink smaller than its <code>flex-basis</code> size.

<br /><h3>Wrapping items and arranging multiple lines</h3>

At this point, you might be thinking *"What if I don't want to shrink my items, and I just want them to wrap to a new line if the container is too small to fit them on the same row?"*.

Great question! We can add a <code>flex-wrap</code> property to our container.

<br /><h4><strong>flex-wrap</strong></h4>

![](/images/2017/02/wrap5.png)

You might suspect in the above image that the items are stretching vertically to fill the empty space in the container. If so, you're right! There is no <code>align-items</code> property set in the container's CSS, which means it defaults to <code>stretch</code>. Similarly, the items are stretching horizontally because I've set <code>flex-grow: 1;</code> in the CSS of the item class.

Let's say we didn't want our items to stretch vertically. In the container's CSS, let's set <code>align-items: flex-start;</code>:

![](/images/2017/02/ai1.png)

BLERGH! What's with the white space between the rows? That's not what I wanted. Don't worry... we'll fix that in the next section.

{% blockquote %}
<code>flex-wrap</code> summary:
  - Add this property to your flex container
  - Use it to control how items wrap (or don't wrap) to the next line when they won't fit in their parent container
  - Some useful values:
    - no-wrap (default)
    - wrap
    - wrap-reverse
  - more info: http://www.w3schools.com/cssref/css3_pr_flex-wrap.asp
{% endblockquote %}


<br /><h4><strong>align-content</strong></h4>

So in the above example, there is some white space between the first row (with the A and B divs) and the second row (with the C and D divs).

This is handled by <code>align-content</code>, which has a default value of <code>stretch</code>, meaning that the wrapped lines will stretch to fit the height of the container. Let's get rid of the space between the rows by adding <code>align-content: flex-start;</code> to our container's CSS:

![](/images/2017/02/ac4.png)

Now our rows are all bunched up at the top of the container, instead of being spread out vertically. If we changed it to <code>align-content: flex-end</code>, the rows would be bunched up at the bottom of the container.

<code>align-content</code> is just like <code>justify-content</code>, except that the latter handles spacing along the main axis, whereas <code>align-content</code> handles spacing along the cross axis.

{% blockquote %}
<code>align-content</code> summary:
  - Add this property to your flex container
  - Use it to control the display of the new rows of items created by wrapping
  - It only works when we have multiple lines of items (i.e. when your content wraps)
  - Some useful values:
    - stretch (default)
    - flex-start
    - center
    - flex-end
    - space-between
    - space-around
  - more info: http://www.w3schools.com/cssref/css3_pr_align-content.asp
{% endblockquote %}

<br /><h3>Go forth and FLEX!</h3>

Well, that's the end of part 2! Hopefully, that's given you an insight into the very cool things that Flexbox can do, and how much easier it can make your life when doing CSS.

<br /><h3>Part 3???</h3>

I **may** do a part 3 on Flexbox soon, but in the meantime, here are some very good resources for you to learn and practice Flexbox:
- Wes Bos' AWESOME (and free) Flexbox course, <a href="https://flexbox.io/">What the Flexbox?</a>
- <a href="flexboxfroggy.com">FlexboxFroggy</a>, a fun game to practice the basics of aligning and positioning content
- <a href="https://css-tricks.com/snippets/css/a-guide-to-flexbox/">A Complete Guide to Flexbox</a>, from css-tricks.com

