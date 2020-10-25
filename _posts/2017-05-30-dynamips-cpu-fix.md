---
layout: post
title: "Dynamips at 100% CPU Usage Fix"
description: "Adjusting Dynamips CPU usage in an virtual environment."
date: 2017-05-30 01:00 +0700
tags: 
    - dynamips
    - vmware
#comments: false
#author: Nazariy Markuta
#share: true
image: /assets/img/dynamips-cpu-fix/
---

While working on a virtual pentest lab in VMWare Fusion. I had the desire of emulating a Cisco router device on a virtual network. The tool I used **Dynamics** did exactly that. However, it was eating up all CPU resources, making other guests almost unusable.


Below shows VMware Fusion process running in Activity Monitor:

![VMware Fusion CPU]({{page.image}}/vmware-cpu.png)


## Fix
Open the Dynagen Management console `dynagen /opt/config.net` and run `idlepc get R1` (R1 is the name of the router) which will calculate a better Idle PC Value for the current guest. This may take a few seconds and should present you with the following:

![dynagen management console]({{page.image}}/dynagen-cpu.png)

As screenshot reads, select a number best suited for your system hinted by an (**\***).
This change will apply for the current session only, meaning it will have to be set again once the program restarts.

## Persistent Change
To make changes persistent, edit your configuration file e.g. `/opt/config.net` and include the `idlepc` keyword and the value calculated previously, in my case:

```
# Simple lab

[localhost]

    [[7200]]
    image = /opt/7200-images/c7200-jk9o3s-mz.124-25d.image
    npe = npe-400
    ram = 256
    # keep CPU usage below 100%.
    idlepc = 0x6079ca5c

    [[ROUTER R1]]
    f0/0 = NIO_linux_eth:eth0
    f1/0 = NIO_linux_eth:eth1
```



I'll post the full workings of my virtual network architecture in the near future.
