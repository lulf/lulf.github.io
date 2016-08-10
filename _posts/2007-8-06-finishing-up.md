---
layout: post
title: "Finishing up"
author: Ulf Lilleengen
categories: freebsd
---
The last couple of weeks I've tested and  done bugfixing and cleanup of gvinum code. I refactored some parts to make the code belong where it seems logical. I also implemented growing for striped plexes, but that was quite easy since I could reuse most of the code for growing RAID-5 plexes.&Acirc;&nbsp;Unfortunately I was sick for a week and unable to work.

What remains now is to do more testing (can't get enough), and write and update documenatation on gvinum. I have updated patches for gvinum at http://folk.ntnu.no/lulf/patches/freebsd/gvinum for both RELENG_6 and CURRENT. I appreciate reports from brave users who tries it out, even if it works :)

Also, I created a new perforce-branch called gvinum_cache. I've currently implemented a read/write-cache to check if this would give much speed-up for gvinum. It's not very nice for reliability, but could be an option for those who want better performance. Anyway, I'll update more on this later.
