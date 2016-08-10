---
layout: post
title: "Sometimes ports make me cry"
author: Ulf Lilleengen
categories: freebsd rant
---
I guess I'm not the typical FreeBSD user, because I do not enjoy using ports
much. Mainly this is because I also use it as a desktop. On a powerful server or
workstation, ports is fine. It's super flexible and everything works quite
well. And kudos to all people working on updating and making improvements to it.

However, using ports on my laptop really makes me cry. Why? If I want to install
a port, I have to keep a ports tree on my laptop and actually compile
everything. Since I have a pretty weak laptop in terms of processing power, this
takes ages. But of course, I can install packages! The thing with packages,
however, is that it works really well for a release, but when upgrading later
on, I always end up in trouble if I try to use the official FreeBSD packages.

First of all, the package sets following each release gets outdated quickly.
Second, if I want to update my packages without using ports I get into trouble.
There is no real package upgrade tool that I know of, but I can install
portupgrade if I want to, because it has a fancy -PP options, telling it to use
packages only. But there are issues with this: portupgrade seems to require that
you have a ports tree to work. In addition, when you have the ports tree,
portupgrade will look for packages matching the exact version that is in ports,
and if the package server does not happen to have the same ports tree as you
(only one commit updating a port can break this), it fails.

So what is the solution for me, besides writing a pkg_upgrade? Having a ports
tinderbox on a different host to build packages for my laptop (I could use
official 8-stable packages for instance, but there always seem to be some
packages missing, and some not built). And the upgrade procedure? Move
/usr/local and /var/db/pkg away, and reinstall packages. It works ok, but
looking at how well this can be handled on other systems, it's a bit silly :/
So, maybe I'll just have to look closer at the pkg_upgrade idea :)

So, on to the constructive part of this rant^Wpost. There is no need to change
everything for this to work better. A pkg_upgrade tool can probably reuse a lot
from the other pkgtools, such as version checking and dependency checking.
However, the hard part is knowing what version to get from the servers. Luckily,
the Latest/ directory contains unversioned tarballs of packages that can be
examined to get their version. But again, this requires one to get the packages
first in order to examine it. Not very bandwidth-friendly.  I think a simple 
approach would be to keep a version list together with the packages, which could
be used by pkg_upgrade to check if any new version of a package exists (much
like INDEX in /usr/ports I guess). I haven't thought about the hardest question
yet: how to handle dependencies and package renaming, but I would think one
could allow specifying this in the same file.

Update: As i was working against my local package repository, I did not notice
that the official package repositories actually contains the INDEX file from the
ports tree where the packages are built.

I also think the package building procedures could be changed, because somehow,
there are always packages missing (at least several gnome packages last time I
tried). I do not know much about this though, but I would advocate for a system
where a package was rebuilt on all architectures and supported releases once a
commit was made to the affecting port.

There, I feel better now :)
