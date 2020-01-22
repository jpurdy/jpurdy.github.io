---
layout: post
title: Create a bootable CentOS USB drive with a Mac (OS X) for a PC
date: 2014-05-23 07:07:07 +0300
description: Bootable CentOS USB drive using MacOS
img: 2014-05-23-1.jpg
fig-caption:
tags: [Mac, Linux]
---
1. Visit Centos’ web page, https://www.centos.org/download/, and download the iso image you’d like to boot from.
2. When the download has completed, open up terminal and use ‘hditutil’ to convert the *.iso to an *.img file (specifically, a UDIF read/write image).
```
$hdiutil convert -format UDRW -o target.img CentOS-7.0-1406-x86_64-Everything.iso
    Reading Master Boot Record (MBR : 0)…
    Reading CentOS 7 x86_64 (Apple_ISO : 1)…
    Reading (Type EF : 2)…
    Reading CentOS 7 x86_64 (Apple_ISO : 3)…
    …………………………………………………………………….
    Elapsed Time: 33.590s
    Speed: 200.5Mbytes/sec
    Savings: 0.0%
    created: /tmp/target.img.dmg
```
3. Use the ‘dd’ utility to copy the iso to your USB drive:
```
$ diskutil list
        /dev/disk0
        #: TYPE NAME SIZE IDENTIFIER
        0: GUID_partition_scheme *121.3 GB disk0
        1: EFI EFI 209.7 MB disk0s1
        2: Apple_HFS Macintosh HD 120.5 GB disk0s2
        3: Apple_Boot Recovery HD 650.0 MB disk0s3
        /dev/disk1
        #: TYPE NAME SIZE IDENTIFIER
        0: FDisk_partition_scheme *31.9 GB disk1
        1: DOS_FAT_32 NO NAME 31.9 GB disk1s1
        /dev/disk2
        #: TYPE NAME SIZE IDENTIFIER
        0: CentOS_7.0_Final *4.5 GB disk2
$ diskutil unmountDisk /dev/disk1**
        Unmount of all volumes on disk1 was successful
$ diskutil unmountDisk /dev/disk2**
        Unmount of all volumes on disk2 was successful
$ time sudo dd if=target.img.dmg of=/dev/disk1 bs=1m**
        Password:
        4261+0 records in
        4261+0 records out
        4467982336 bytes transferred in 1215.483272 secs (3675890 bytes/sec)
```
4. You should be done! Boot from the USB drive on your target machine.
