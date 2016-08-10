---
layout: post
title: "Bsdadminscripts to the rescue"
author: Ulf Lilleengen
categories: freebsd
---
I just learned of sysutils/bsdadminscripts after my previous post about how hard
it was to use packages only in FreeBSD. Well, I think I found a partial solution
to my problem, as the bsdadminscripts port contains a pkg_upgrade utility, which
is able to update your system without a ports tree available, as long as the
INDEX file exist on the packages server.

I now use this in combination with my port tinderbox, building the packages I
want for my laptop. Then I generate the INDEX file in the tinderbox ports tree,
and put it into the packages folder of the tinderbox. Voila! I can now use
pkg_upgrade -a, and all packages are upgraded to the latest version.

There are a few things that I think can be improved: Have the tinderbox scripts
automatically generate the INDEX file and putting it into the packages directory
with a simple command or just do it on an update of the ports tree. The other
thing is what I mentioned in my previous post about keeping the official
packages properly up to date.
