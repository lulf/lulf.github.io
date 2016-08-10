---
layout: post
title: "Gvinum imported"
author: Ulf Lilleengen
categories: freebsd
---
Last weekend I imported gvinum into HEAD, and I hope many users (and old users)
of gvinum will try it out, as it have some nice improvements. Moving it into
HEAD now, means it also will become part of 8.0-RELEASE which is coming later
this year, and since it is a lot of changes, the intention is to have it in HEAD
now for a while before the release process begins. Among the most interesting
updates for users are:

* Support more of the old vinum command set
* Less panics :)
* Rebuilding and synchronizing plexes can be done while mounted.
* Support for growing striped or raid5 plexes while mounted, meaning that you
  can just add a new disk to your gvinum configuration, and grow it to cover the
  new disk.
