---
layout: post
title: "Growing up"
author: Ulf Lilleengen
categories: freebsd
---
Since last post I haven't really done that much do gvinum, but a few things.
<ul>
	<li>I added a few automated test-scripts to check if a volume behaves properly</li>
	<li>Go through test-plan and make sure that gvinum passes the tests.</li>
	<li>I've been thinking a lot on how to best implement growing RAID-5 plexes.</li>
	<li>I've implemented growing of RAID-5 plexes.</li>
</ul>
Now, the first and second points are quite boring to do, but I had to do it. Now the last points were trickier, since I didn't really know where I should start. Finally I decided the best way was to let the plex overwrite itself! A more detalied explanation can be found in the TODO of my perforce branch.  I need to test the implementation a bit now. Other than that, I've been a bit lazy on my own work this week, and tried to help other students with reviews etc.
