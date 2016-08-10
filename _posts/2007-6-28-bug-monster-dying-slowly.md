---
layout: post
title: "Bug monster dying slowly"
author: Ulf Lilleengen
categories: freebsd
---
<p class="line874">Finally, an update on what I've been doing since the last time. This time I have a lot of small changes that have been done: <span class="anchor"></span></p>

<ul>
	<li>Implement initialization of RAID5-Arrays. This basically writes zeros over everything and makes sure parity is correct. <span class="anchor"></span><span class="anchor"></span></li>
<li class="gap">Fix a bug with mirror code. The  length of the completed requests got  doubled if you have a  mirror with two plexes, tripled if you have  a mirror with three plexes etc.<span class="anchor"></span><span class="anchor"></span></li>
<li class="gap">When a mirrored plexes are syncing, all requests after and including the first _write-request_ are delayed until syncing is finished. <span class="anchor"></span><span class="anchor"></span></li>
<li class="gap">Allow rebuilding a RAID-5 array while it is in use (e.g. mounted). Delay requests that are in conflict with the rebuild, but allow requests on the already rebuilt part to be run. <span class="anchor"></span><span class="anchor"></span></li>
<li class="gap">Allow subdisks to come up automagically after rebuild. <span class="anchor"></span><span class="anchor"></span></li>
<li class="gap">Allow stripesizes not divideable by the subdisk size. A regression in the new gvinum code prevented this. <span class="anchor"></span><span class="anchor"></span></li>
<li class="gap">Modify the event system to contain two intmax_t fields, so we won't have to allocate/deallocate pointers all the time when passing args to gv_post_event. <span class="anchor"></span><span class="anchor"></span></li>
<li class="gap">Add support for the rename and move commands to new gvinum. The code has been rewritten for the new gvinum.</li>
<li class="gap">Fix a bug in the code for degraded writes to a RAID5-array, where only zeros were written.</li>
<li class="gap">Other minor bug/style fixes. <span class="anchor"></span><span class="anchor"></span></li>
</ul>
Next, I'm going to implement concat/stripe/mirror functionality. I already have some code from previous work I did, so I just need to adapt it to new gvinum, as well as change some ugly parts. There are some small facade-changes left, but I will do this after the last of the original vinum features is completed. Also, I will try write a nice status report, and get a testable patch out by the time the reports are finished.
