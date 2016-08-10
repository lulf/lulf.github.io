---
layout: post
title: "Locale fix"
author: Ulf Lilleengen
categories: freebsd
---
After looking for a long time as to why my default locale in gnome changed after
a recent upgrade, I finally found out where to change the locale setting. The
problem was that gnome did not seem to pick up my system locale settings, and
the norwegian characters in my terminal came up as question marks.

As the gnome login manager (gdm) got rewritten, there is now no way to change
this locale at the login screen unless it was picked up by gdm. But, as always,
reading the documentation helps. After reading

http://library.gnome.org/admin/gdm/2.28/configuration.html.en

I discovered that I could just edit

    ~/.dmrc

and write this:

    [Desktop]
    Language=en_US.UTF-8
    Layout=no

to set the correct locale!
