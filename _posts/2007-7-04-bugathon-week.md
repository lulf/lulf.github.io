---
layout: post
title: "Bugathon week"
author: Ulf Lilleengen
categories: freebsd
---
Since last post, there has been many small bugfixes to gvinum. After some debating with myself on how I should implement concat/stripe/mirror, I think I got it pretty much right. The event system changed gvinum a lot, so I had to rewrite most of the code I already had on this.

I have done a lot of testing this week, and I made a test plan that I'm going to follow. Hopefully, I'll also be able to create som automatic tests for this.

I've even been a good boy and updated the gvinum manpage! I added some examples to the manpage as well, so that it's easier to get into gvinum for inexperienced users (not sure if we want gvinum to live even longer, but :) ).

A lot of small problems with weird states being set was also fixed, since this can be very confusing if you havent used gvinum much.

What I'd like to do next, is create a set of testscripts that I can use to test quickly and easily with. I also noticed that it would be nice to have a similar command like 'mirror' for RAID-5 volumes. This could be used like this: 'gvinum raid5 &lt;disk1&gt; &lt;disk2&gt; &lt;disk3&gt;'. Other than that, I've started to think on how I'm going to implement raid-5 resizing and other goals in my proposal.
