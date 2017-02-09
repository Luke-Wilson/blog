---
title: Intro to Flexbox Pt2
tags:
- CSS
- Flexbox
---



**align-items** is

align-items: stretch(default), flex-start, flex-end, center, baseline

NOTE: 'baseline' is pretty cool. It looks at the bottom of the text within each flex item and makes sure they all align with each other. So if you had different-sized fonts in different items, then you could align all of their baselines.



----------------------------------------
Width
----------------------------------------
If you set a width on flex items, Flexbox will try to work with the width you give it. But if it doesn't work out it will evenly distribute the width throughout the children OR you can use flex-wrap property on the flex container. If you set the container to wrap, then the width value you gave the items will work and each item will just wrap to the next line.

flex-wrap: nowrap (default), wrap, wrap-reverse.


----------------------------------------

----------------------------------------
align-content only works when we have multiple lines of elements (i.e. when your content wraps)

The width of your flex items by default is 'auto', meaning it will be as wide as it needs to be because of the text inside it (?).

**flex** tells the item how it should proportionally scale itself up (or down) when there is too much (or not enough) space, as well as setting a flex-basis width (or other value).

So when we give all our flex items a property of "flex" with the same value (e.g. 1), we're saying "WHEN THERE IS EXTRA SPACE...", that space should be divvied up evenly for all flex items. If however, we gave one item a flex value of '2', then "WHEN THERE IS EXTRA SPACE" that item would receive twice as much extra space as any item with a flex value of '1'.

{% codeblock %}
.item {
  flex: 1 5 400px;
}
{% endcodeblock %}
is actually shorthand for:
{% codeblock %}
.item {
  flex-basis: 400px;
  flex-grow: 1;
  flex-shrink: 5;
}
{% endcodeblock %}

**flex-grow**: (default is 0) When we have extra space, how should we divide it amongst the items on the same line? If we set an item's flex-grow to '3', then it will be given three times as much of the extra space than those items with a flex-grow value of 1. An item with the default flex-grow value (0) will just stay at its flex-basis size when there is extra space... it won't grow.

**flex-shrink**: (default is 1) When we don't have enough space, how do we shrink an item down? (i.e. Will it shrink if the flex items on that line overflow the flex container, and if so, by what factor?). If we set an item's flex-shrink to '2', then it will shrink twice as much as an item with flex-shrink 1. An item with a flex-shrink value of 0 will not shrink smaller than its flex-basis size. If you need to do this, you might want to also think about using flex-wrap in the flex container, or even using <code>overflow: auto</code>.

**flex-basis**: (default is auto) <code>flex-basis</code> lets you set the initial size of a flex item before any additional space is added to it via flex-grow (or any space is removed from it via flex-shrink). It can either be an absolute value, or a percentage based on its container's width (or more specifically, the container's length along the main axis).




flex (or flex-grow, flex-shrink and flex-basis)

"The whole of point of flexbox is that if you have too much extra space, or not enough space, your layout won't break. It will intelligently figure out where everything should go".








Senegal flag
https://jsfiddle.net/lukewilson/gsoLtagu/4/




Resources

What the Flexbox (course): https://flexbox.io
FlexboxFroggy: http://flexboxfroggy.com/





Prefixing
- You need to do vendor prefixing to ensure your code supports the most browsers possible.
- Use Autoprefixer for this: https://github.com/postcss/autoprefixer
