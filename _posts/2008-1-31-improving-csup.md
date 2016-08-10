---
layout: post
title: "Improving csup"
author: Ulf Lilleengen
categories: freebsd
---
&nbsp;It's been a while. Partially because I've become a FreeBSD committer and had more productive stuff to do than writing in my weblog, and partially because my account was disabled after Google Summer of Code (Also, thanks to Google for SoC).

&nbsp;Since last time, I've been working on getting my gvinum work of this summer into the tree, but since this has to be reviewed before going into the tree, and 7.0-RELEASE is much more important right now, it's sort of on hold. In the meantime I've been working on implementing CVSmode for csup. This is something I've been meaning to do for a long time, but never found the big motivation until my exam-period before last christmas. So, I'll tell you a bit of what I've done here. For those of you unfamiliar with cvsup,&nbsp;it's a network CVS file synchronization tool which is heavily used in FreeBSD. However, it is written in Modula-3 and is therefore not very easy to maintain and it doesn't integrate into the FreeBSD base system very well. So, Maxime Henrion started a C rewrite of cvsup called csup.

First, a bit on how csup works (or the cvsup protocol). The client runs three threads performing these tasks:
<ul>
	<li>The lister, which examines the clients files, and sends information about them to the server.</li>
	<li>The detailer, which recieves commands from server on what needs to be done ("this file needs updating, send me the details of it's revisions).</li>
	<li>The updater, which recieves the actual updates from the server ("add this delta to the RCSfile").</li>
</ul>
More details on how the protocol works can be found on <a href="http://www.cvsup.org/howsofast.html">http://www.cvsup.org/howsofast.html</a>

&nbsp;So, what is CVSmode anyway? In csups normal operation, csup requests the files from a specific branch, called checkout mode. This is the typical way a user would use csup, fetching the src-tree for RELENG_7 for instance. However, a developer would often like to have the FreeBSD CVS repository on his local machine, and this is where CVSmode plays a part. CVSmode means that csup will recieve the entire CVS repository, and also fetch updates to the actual RCSfiles. So far, csup does only support the checkout mode.

So, what's needed for CVSMode to work?
<ol>
	<li>Support for the protocol, so the client is able to not only act correctly on the commands from the server, but also respond correctly. This involves modifying the detailer and the updater part of csup.&nbsp; This part needs to be a bit cleaned up right now, but is in a working state.</li>
	<li>Correctly parse RCSFiles. Firstly, I made a lexer with flex and parser with yacc. Then I found out I needed reentrancy, and started using bison. After realizing using bison for this wasn't really nice since bison wasn't in base, I rewrote the parser in C.</li>
	<li>The ability to update RCSfiles. This required a RCSfile interface. This interface is used by both the parser and the updater, to import and edit RCSfiles. Writing this interface is probably what has taken most of my time.</li>
	<li>Writing the RCSFiles out with the new updates. This is done internally by the RCSfile implementation.</li>
</ol>
So, this is what I've been working on implementing the last month or two. And I have the most parts working. What's missing is a crucial part of (4). To write out the new RCSFiles to disk, a correct algorithm to apply diffs _and_ reverse diffs is needed. The algorithm for applying diff was already created by csups author, but the reverse diff algorithm is a bit different. The last week or so, I've been studying the algorithm used in cvsup, and I've started to implement something similar although a bit different in it's implementation. So, hopefully I'll have this work pretty soon, at least before people start switching over to some new version control system :)
