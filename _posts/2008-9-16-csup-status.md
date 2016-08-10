---
layout: post
title: "Csup status"
author: Ulf Lilleengen
categories: freebsd
---
Finally, I have been able to resolve all current known issues with cvsmode support in csup. I just sent out a new announcement with the patch, and I hope to get some more testing and perhaps some reviews soon, but it is a big patch and few people are familiar with the code base.

Remaining issues with the patch is support for using the status file when reading (but this is not critical at all), as well as rsync support (which is only significant for a few files in the freebsd repository).

I hope as many as possible are able to test it:

http://people.freebsd.org/~lulf/patches/csup/csup_09_16_2008.diff
