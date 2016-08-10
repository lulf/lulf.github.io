---
layout: post
title: "Subdisks nodw live happy in raid5 town"
author: Ulf Lilleengen
categories: freebsd
---
So, finally the exams are over, and I've been able to work sort of full-time on my project the last days. What I've done is (a bit technical this time perhaps, but this stuff tends to&Acirc;&nbsp;become&Acirc;&nbsp;that):

Implemented attach/detach routines. This makes it possible to attach a subdisk/plex to a plex/volume, or detach a plex/subdisk from a volume/plex. The detach routine makes sure all connections between the objects are broken correctly, and only if it's possible (unless forced ofcourse). The attach routine makes sure the objects are correctly connected together again, and that a plex that misses subdisks includes them in the previous size when calculating the new size (so we don't get wrong sizes on the plexes).

Tested rebuild of degraded plexes. The detach/attach routines enabled me to check if the rebuild of a degraded raid5 plex could work. And it did! This means (and this is something that I really missed in old gvinum), that when a drive fails, you can detach the failed subdisk, create a new subdisk on the plex (and it will check if the plex "misses" a subdisk), and then use 'start &lt;plexname&gt;' to rebuild the plex (The state of the plex must be degraded and the subdisk you wish to rebuild must be stale) and you're good to go!

Bugfixes.

Implement syncing of plexes.  This means one can now add a mirror to a volume, and have the new plex to be synced from the original. After a couple of tests, it seems to work, but I did get a bug I need to reproduce.

I&Acirc;&nbsp;also&Acirc;&nbsp;discovered&Acirc;&nbsp;some&Acirc;&nbsp;bugs&Acirc;&nbsp;regarding&Acirc;&nbsp;mirrored&Acirc;&nbsp;plexes&Acirc;&nbsp;that&Acirc;&nbsp;I&Acirc;&nbsp;will&Acirc;&nbsp;address&Acirc;&nbsp;in
the&Acirc;&nbsp;near&Acirc;&nbsp;future.&Acirc;&nbsp;This&Acirc;&nbsp;probably&Acirc;&nbsp;came&Acirc;&nbsp;with&Acirc;&nbsp;the&Acirc;&nbsp;change&Acirc;&nbsp;with&Acirc;&nbsp;the&Acirc;&nbsp;new&Acirc;&nbsp;gvinum
event&Acirc;&nbsp;system.

Next on my schedule is to hunt some weird state bugs where the state is not correctly set, as well as the mirrored plex problems I've seen. Also, I need to guarantee that a plex sync is up-to-date (that no data is written to the synced plex in the meantime).
