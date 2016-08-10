---
layout: post
title: "Raid5 improvements"
author: Ulf Lilleengen
categories: freebsd
---
The last couple of weeks I had to practice for some exams. In other words, a great time for coding :)

This week I've been working on making RAID5 parity rebuild work. This includes user initiated rebuild/check, as well as rebuilding a degraded plex during plex initialization. (This is a vital feature, since if a drive fails, one must be able to rebuild the plex with the new drive. I have not been able to test this enough yet because I need the attach/detach routines to do it. So instead of continuing and getting the initalization/synchronization-routines in, I will implement attach/detach next, which should be quick since I already have some old code for it.
