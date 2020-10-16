---
layout: post
title: Bad UEFI implementation Workaround
description: Getting round poor UEFI implentation support for booting into Debian Stretch.
date: 2017-06-18 01:00 +0700
tags: 
    - uefi
    - boot issue
    - debian
#comments: false
#author: Nazariy Markuta
#share: true
redirect_from: "/Bad-UEFI-Implementation-Workaround/"
---
I decided to install the latest stable branch of Debian Stretch on a budget laptop (Toshiba Satellite C50-B-14D) bought in 2015. Its minimal specs was perfect for Linux. Installation image used was `debian-9.0.0-amd64-netinst.iso`. Once the installation process finished and I restarted my system, it would not recognise grub or any boot partition.

### Solution

> Rename folder and filename **/EFI/debian/grubx64.efi** to **/EFI/boot/bootx64.efi**     
> *Read more: [https://wiki.debian.org/UEFI#Booting_a_UEFI_machine_normally](https://wiki.debian.org/UEFI#Booting_a_UEFI_machine_normally)*

With a different filename the UEFI implementation recognised and booted successfully. This might not work on all manunfacturers.

### Commands with Mac OS EL Capitan
Below lists terminal commands used to quickly mount disks and rename files and folders using an external hard disk connected. Disk names may vary to yours.
```
$ diskutil list
$ diskutil mount /dev/disk2s1
$ cd /Volumes/NO\ NAME/EFI/
$ mv debian boot
$ cd boot
$ mv grubx64.efi bootx64.efi
$ sync
$ diskutil umount /dev/disk2s1
```
