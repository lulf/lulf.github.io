---
layout: post
title: "Hunting bugs"
author: Ulf Lilleengen
categories: freebsd
---
More status updates... I've been fixing many small gvinum bugs the last couple of weeks:
<ul>
	<li>The state of gvinum objects were changed after reloading. This meant that objects got the wrong state when gvinum was brought up.</li>
	<li>Made gvinum always use the most recent configuration it finds when setting object states.</li>
	<li>Make sure the newest drive is always the newest, and not the first in the drivelist, as was previously assumed.</li>
	<li>Add "growable"-state to be used when a plex is ready to be grown.</li>
	<li>Allow a plex to be rebuilt even though it's also growable.</li>
	<li>Do not change the size of the volume until the plex is completely grown.</li>
	<li>Add status of growing and rebuild of a plex in the list output.</li>
	<li>Prevent rebuild to take over the I/O system increasing access-count at the start and end of the rebuild.</li>
</ul>
Probably a couple of other fixes as well. Also, I've updated the vinum-examples page in the handbook to reflect new features and more practical examples. I've posted a "call for testers" on current@, arch@ and geom@, and have received some response from people who are willing to help me test. Thanks to them. I've uploaded the code-sample that I'll be delivering to google here: http://folk.ntnu.no/lulf/gvinum_soc2007.tar.gz
