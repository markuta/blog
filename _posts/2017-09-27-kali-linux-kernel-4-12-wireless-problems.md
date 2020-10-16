---
layout: post
title: Kali Linux Kernel 4.12 Wireless Problems
date: 2017-09-27 09:45:47 +07:00
description: The recent Kali Linux Kernel update version 4.12.12-2kali1 completely broke wireless functionality resulting in poor performance and range issues. This post shows how to quickly resolve the wireless issues by downgrading back to the previous kernel version 4.11.6-1kali1.
tags: 
    - kali linux
    - kernel
    - wireless
image: /assets/img/kali-linux-kernel-4-12-wireless-problems
---
#### UPDATE: Kali Team have migrated to a new Kernel Version
I've tested the new kernel **4.12.13-1kali2** and can confirm it fixes issues with my wireless card. I'd advise people to update their packages and use this kernel. I'll keep this page as it's still a useful guide on how to downgrade for falling back on an other kernel. More info available: [https://pkg.kali.org/pkg/linux](https://pkg.kali.org/pkg/linux)

As you may of heard the recent **4.12.6-1kali1** kernel version broke functionality on most wireless devices, resulting in serious performance and range issues making devices almost unusable. I also noticed my wireless card getting rather hot warm for my liking.

Here are some of the error messages in **dmesg** produced on the current Kernel while in monitoring mode:

![alt text]({{page.image}}/kali-linux-dmesg-wireless-errors.png "Kali Linux Kernel 4.12 Wireless Errirs")

The range also drops dramatically, where I couldn't even monitor traffic 3 meters away from my access point.

### How to Downgrade back to Kernel 4.11

Here is a quick way of installing the previous *working* kernel **linux 4.11.6-1kali1** that'll resolve wireless device issues until a patch is released. The same version is used in the [AWUS052NH card review](https://markuta.com/alfa-awus052nh-review/).

1. Head over to: [https://http.kali.org/kali/pool/main/l/linux/](https://http.kali.org/kali/pool/main/l/linux/)
2. Find **4.11.6-1kali1** and download your architecture for example 64-Bit Intel or AMD CPUs would be: `linux-image-4.11.0-kali1-amd64_4.11.6-1kali1_amd64.deb`
3. Install the .deb package with **dpkg**:
`dpkg -i linux-image-4.11.0-kali1-amd64_4.11.6-1kali1_amd64.deb`

4. Reboot the system. During the initial boot process select `Advanced options` from the GRUB menu and the kernel version we've just installed:
![alt text]({{page.image}}/kali-linux-kernel-4.11.png "Kali Linux Kernel 4.11")

After booting the wireless device should work as before the update.
