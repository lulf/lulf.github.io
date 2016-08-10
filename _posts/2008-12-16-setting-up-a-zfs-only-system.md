---
layout: post
title: "Setting up a zfs only system"
author: Ulf Lilleengen
categories: freebsd
---
After loader support for ZFS was imported into FreeBSD around a month ago, I've been thinking of installing a ZFS-only system on my laptop. I also decided to try out using the GPT layout instead of using disklabels etc.

The first thing I started with was to grab a snapshot of FreeBSD CURRENT. However, I discovered that the loader doesn't support ZFS, so you have to build your own FreeBSD cd in order to install a working loader! Look in src/release/Makefile and src/release/i386/mkisoimages.sh for how to do this. Since sysinstall doesn't support setting up ZFS etc, it can't be used, so one have to use the Fixit environment on the FreeBSD install cd to set it up. I started out by removing the existing partition table on the disk (just writing zeros to the start of the disk will do). 

Then, the next step was to setup the GPT with the partitions that I wanted to have. Using gpt in FreeBSD, one should create one partition to contain the initial gptzfsboot loader. In addition, I wanted a swap partition, as well as a partition to use for a zpool for the whole system.

To setup the GPT, I used gpart(8) and looked at examples from the man-page. The first thing to do is to setup the GPT partition scheme, first by creating the partition table, and then add the appropriate partitions.

    gpart create -s GPT ad4
    gpart add -b 34 -s 128 -t freebsd-boot ad4
    gpart add -b 162 -s 5242880 -t freebsd-swap ad4
    gpart add -b 5243042 -s 125829120 -t freebsd-zfs ad4
    gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ad4

This creates the initial GPT, and adds three partitions. The first partition contains the gptzfsboot loader which is able to recognize and load the loader from a zfs partition. The second partition is the swap partition (I used 2.5 GB for swap in this case). The third partition is the partition containing the zpool (60GB). Sizes and offsets are specified in sectors (1 sector is typically 512 bytes). The last command puts the needed bootcode into ad4p1 (freebsd-boot).

Having setup the partitions, the hardest part should be done. As we are in the fixit environment, we can now create the zpool as well.

    zpool create data /dev/ad4p3

The zpool should now be up and running. I then decided to create the different filesystems i wanted to have in this pool. I created /usr, /home and /var (I use tmpfs for /tmp).

Then, freebsd must be installed on the system. I did this by copying all folders from /dist in the fixit environment into the zpool. In addition, the /dev folder have to be created. For better details on this, you can follow (http://wiki.freebsd.org/AppleMacbook) At least /dist/boot should be copied in order to be able to boot.

Then, the boot have to be setup. First, boot/loader.conf have to contain:

    zfs_load="YES"
    vfs.root.mountfrom="zfs:data"


Any additional filesystems or swap has to be entered into etc/fstab, in my case:

    /dev/ad4p2 none swap sw 0 0

I also entered the following into etc/rc.conf

    zfs_enable="YES"


In addition, boot/zfs/zpool.cache has to exist in order to be able to let the zpool be imported automatically when zfs loads on system boot. To do this, I had to:

    mkdir /boot/zfs
    zpool export data && zpool import data

In order to make /boot/zfs/zpool.cache get populated in the Fixit environment. Then, I copied zpool.cache to boot/zfs on the zpool:

    cp /boot/zfs/zpool.cache /data/boot/zfs

Finally, a basic system should be installed.The last ting to do is to unmount the filesystem(s) and set a few properties:

    zfs set mountpoint=legacy data
    zfs set mountpoint=/usr data/usr
    zfs set mountpoint=/var data/var
    zfs set mountpoint=/home data/home
    zpool set bootfs=data data

To get all the quirks right, such as permissions etc, you should to a real install with making world or using sysinstall when booted into the system. Reboot, and you might be as lucky as me and boot into your ZFS-only system :) For further information, take a look at:

<a href="http://wiki.freebsd.org/ZFSOnRoot">http://wiki.freebsd.org/ZFSOnRoot</a>
which contains some information on how to use ZFS as root, but by booting from ufs and:
<a href="http://wiki.freebsd.org/AppleMacbook">http://wiki.freebsd.org/AppleMacbook</a>
which has a nice section on setting up the zpool in a Fixit environment.

Update:

When rebuilding FreeBSD after this type of install, it's also important that you build with <span class="caps">LOADER_ZFS_SUPPORT=YES</span> in order for the loader to be able to read zpools.
