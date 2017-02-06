---
title: Media queries and responsive design
date: 2017-02-03 12:30:45
tags:
- CSS
- responsive design
---

![Responsive design according to Bruce Lee](/images/2017/02/Content-is-like-water.jpg)

CSS is one of my skill sets that I've been meaning to strengthen for a while now. I really love working with design, and while I've had a fair bit of design experience creating web graphics and from my photography, up until now I've focused more on programming and less on design. That said, I LOVE the feeling that comes with building a beautiful page or element.
<!-- more -->
<h3>Not sure what media queries are? Start here.</h3>

Making beautiful pages is great, but with the rise of mobile, beauty is not enough. Users will be accessing your content via a range of different devices. That's where media queries can help.

Media queries allow you to display different layouts and styles for different viewport widths. (There's more than just going by width, but this is probably the most common media query.) So a user on a desktop will see a different design to what a user would see on their iPhone or iPad.

Media queries are conditional statements, so if the media query is true, the corresponding styles are applied.

Many developers used to make breakpoints device-specific, meaning an iPhone 4 would get a breakpoint, an iPhone 5 would get a different breakpoint, an iPad would get a different breakpoint and so on. With so many different devices, that was a pain in the butt.

<h3>Content-based breakpoints</h3>

It's better to choose breakpoints based on your content. The best way to do this is to take a 'mobile first' approach, or as <a href="http://bradfrost.com/blog/mobile/bdconf-stephen-hay-presents-responsive-design-workflow/">Stephen Hay</a> says "Start with the small screen first, then expand until it looks like shit. Time for a breakpoint!".

Designing for mobile first is especially important considering that, out of all the time people spend on the web, <a href="http://www.smartinsights.com/mobile-marketing/mobile-marketing-analytics/mobile-marketing-statistics/">the majority of that time is spent accessing it from a mobile device (phone or tablet)</a>. Below is an example of the 'mobile first' approach:

{% codeblock %}
/* Styles for smallest viewport as well as styles for all screen sizes */
/* (regular CSS without media query) */

/* Media query with a small min-width breakpoint */
@media only screen and (min-width: 480px) {

}

/* A slightly larger viewport's breakpoint */
@media only screen and (min-width: 768px) {

}

/* An even larger viewport's breakpoint */
@media only screen and (min-width: 960px) {

}

/* Large viewports (e.g. TV, widescreens) */
@media only screen and (min-width : 1200px) {

}
{% endcodeblock %}

You can do more with media-queries. For example, if you wanted users using an iPad in landscape mode to be shown a specific style you could specify that using a media query:

{% codeblock %}
/* iPad-sized viewport in portrait */
@media only screen
and (min-device-width : 768px) { /* STYLES GO HERE */}

/* iPad-sized viewport in landscape */
@media only screen
and (min-device-width : 768px)
and (orientation : landscape) { /* STYLES GO HERE */}
{% endcodeblock %}

<h3>Add some breakpoints</h3>

So, following Stephen Hay's advice, when you are designing, just expand your browser width until it starts looking crap. This is where you add a breakpoint! Oh, and since we're taking a mobile-first approach, we just have to use min-width without specifying a max-width.

If we've done everything right, it should mean that our style will be future-proofed from the litany of new and different-sized devices that hit the stores every year, and our designs will always look great at any given width.