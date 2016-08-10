---
layout: post
title: "Using 4k sector drives"
author: Ulf Lilleengen
categories: freebsd
---
I just bought two Western Digital 2 TB disks the other day in order to increase
storage capacity. I was planning on putting a ZFS mirror on them. The other day
I discovered that the disks uses a new drive format called "Advanced Disk
Format". This format basically extends the sector size from 512 to 4096 bytes.

The problem is that the disks report their sector size to be 512 rather than
4096 in order for them to work well with existing operating systems. The issues
with these disks are discussed  [here](http://docs.freebsd.org/cgi/getmsg.cgi?fetch=133982+0+archive/2010/freebsd-hackers/20100815.freebsd-hackers)
and
[here](http://docs.freebsd.org/cgi/getmsg.cgi?fetch=0+0+archive/2010/freebsd-hackers/20100815.freebsd-hackers).

To summarize, this results in two main problems:

1. Partitioning tools operate on 512 bytes "logical" sectors, which may result
in a partition starting at a non-aligned (compared to 4096 bytes) physical
sector. If using partitioning tools that are not updated to align partitions to
4k, a request may cause a write to more than one sector.

2. File systems/disk consumers think the underlying device has a 512 byte sector
size, and issues requests that are below 4096 bytes. For a write request, this
is catastrophic, because in order to write only parts of a block, the disk will
have to read the block and modify the part that changed, before writing it back
to disk (Read-modify-write). 

Dag Erling Smørgrav made a tool to benchmark disk performance using aligned and
misaligned writes (mentioned in his post above (svn co svn://svn.freebsd.org/base/user/des/phybs). Here are the results:

    nobby# ./phybs -w /dev/gpt/storage0
    count    size  offset    step        msec     tps    kBps

    131072    1024       0    4096      131771      16     994
    131072    1024     512    4096      136005      16     963

     65536    2048       0    8192       74762      14    1753
     65536    2048     512    8192       71407      15    1835
     65536    2048    1024    8192       73432      15    1784

     32768    4096       0   16384       20710     130    6328
     32768    4096     512   16384       61987      43    2114
     32768    4096    1024   16384       62719      43    2089
     32768    4096    2048   16384       61089      44    2145

     16384    8192       0   32768       14238     245    9205
     16384    8192     512   32768       53348      65    2456
     16384    8192    1024   32768       52868      66    2479
     16384    8192    2048   32768       50914      68    2574
 
Clearly, using < 4k blocks results in bad performance. Using blocks larger than
4k results in a 3x speedup. 

The way I solved this in FreeBSD was to partition the disk manually with gpart
and set the partition start to a multiple of 8 (8 * 512 = 4096). All partitions
on the disk should start at a sector number that is a multiple of 8.

ZFS uses variable block sizes for its requests, which can pose a problem when
the underlying provider reports a sector size of 512 bytes. In order to override
this, I used gnop(8), which can create a provider on top of another provider
with different characteristics:
	gnop create -o 4096 -S 4096

The -o parameter makes sure that the new provider does not conflict with the
original provider when ZFS tries to detect any filesystems on the disk. The
second parameter sets the sector size of the new parameter to 4096, which makes
sure that all requests going to the disk from ZFS will be in 4k blocks.

For UFS, the default fixed block size is 16k, so there should be no worries
about it using lower block sizes. Moreover, newfs provides a -S parameter, which
overrides the sector size of the underlying provider. I have not tried using UFS
on these disks, but I don't see any reason for it not working.
