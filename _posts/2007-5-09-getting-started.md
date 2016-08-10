---
layout: post
title: "Getting started"
author: Ulf Lilleengen
categories: freebsd
---
Well, there has been some time now to get familiar with the projects and how things are done, although I am already quite familiar with the procedures. Therefore I started a bit earlier on the actual project, since I have a period with finishing exams from now until 8. June.

I've committed some work on the setstate functionality making it possible to be used again after Lukas' rewrite of gvinums event-handling. Setstate now works on subdisks, plexes, volumes and drives. (Only subdisks and drives was supported before).

I have also been thinking of and made error codes to be used internally in gvinum to sort of help me see what happens when debugging etc. Other than that, I've rewritten the resetconfig a bit to prevent (what i think is) a race condition that was introduced with the new event system.

My plan is now to further adapt gvinum code to the new event system. I also have more patches from my previous that need to be integrated.
