---
layout: post
title: "Linuxulator to the rescue"
author: Ulf Lilleengen
categories: freebsd technical
---
As I usually have a few classes at school which requires special software, I
wanted to be able to run some of this software on my own computer, as there are
student versions of some of the software. One of these is
[ModelSim](http://www.model.com/) from MentorGraphics. ModelSim is basically a
simulator for hardware designs, and I use it to simulate
[VHDL](http://en.wikipedia.org/wiki/VHDL). Unfortunately, ModelSim only comes
for Windows, [Linux](http://kernel.org/) and Solaris. As I only run [FreeBSD](http://www.freebsd.org/) on my laptop, no software
for me :( But wait, FreeBSD have the
[linuxulator](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/linuxemu.html)!,
which allows Linux binaries to be run unmodified on a FreeBSD host (It is basically an implementation of Linux syscalls within the FreeBSD kernel). The steps I
needed to go through to install the Linux version of ModelSim was pretty easy.

First of all, one of the emulators/linux_base* ports needs to be installed. I
chose linux_base-fc6, as I'd like the Linux 2.6 support (although I'm not sure
if that is actually needed). After installing the port, a linux userland appears
in /compat/linux. To make sure I don't get any problems with programs needing
procfs, I mount
[linprocfs\(5\)](http://www.freebsd.org/cgi/man.cgi?query=linprocfs&apropos=0&sektion=0&manpath=FreeBSD+8-current&format=html)
as well.

There, easy! Ready to run linux programs. Now, ModelSim comes with its own
installer, which needs a few additional files that you can get at their
web-site. However, programs may depend on additional libraries, and this is IMO
the most tricky part about the linuxulator. In my case, I got some errors
complaining about not finding libXsomething. Luckily, there are a few ports that
you can install for the most common libraries. In this case, I had to install
x11/linux-xorg-libs. Although a very old version, I was able to run the ModelSim
installer and the installed binaries afterwards. Awesome!
