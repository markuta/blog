---
layout: post
title: How to build a Debian MIPS image on QEMU
date: 2018-03-18 01:00 +0700
description: This guide shows how to install and build a MIPS (Big Endian) Debian Stretch (9.4) image for running under QEMU virtualization software. These steps could also be applied to other MIPS architectures.
tags: 
  - mips
  - qemu
  - debian
  - virtualization
#author: Nazariy Markuta
#comments: false
#share: true
image: /assets/img/how-to-build-a-mips-qemu-image-on-debian/
---
How to set up and build your own MIPS big endian or little endian image running under the QEMU emulator. This guide can also be applied to other architectures. For example I'm currently running this in a virtual machine inside another virtual machine on my MacBook Pro. "We Need To Go Deeper" - Dom Cobb, Inception.


![alt text]({{page.image}}/mips-screen.png "MIPS-32 Big Endian")

### Install Package
Since we are only emulating a MIPS system on QEMU, we only require a specific package namely; **qemu-system-mips**. On most Linux distos you can simply install through **apt-get**. This will also install further packages required by QEMU:
```bash
$ sudo apt-get install qemu-system-mips
```

The exact version used: _QEMU emulator version 2.8.1(Debian 1:2.8+dfsg-6+deb9u3)_.

### Download files
There are two versions of the MIPS-32 (Big Endian); **Malta** and **Octeon**. This guide will be using the Malta version. Although it's almost the exact same process for Octeon with a few minor option differences.

> The Kernel filename may differ to one listed below. As the Debian team provides newer releases and updates, the filename will change over time. Link to [Latest Stable](http://ftp.debian.org/debian/dists/stable/main/installer-mips/current/images/malta/netboot/){: target="_blank" __}.

Download both the installer and boot files from stable release:

* Installer (**initrd.gz**) ~21MB:
```bash
$ wget http://ftp.debian.org/debian/dists/stable/main/installer-mips/current/images/malta/netboot/initrd.gz
```
* Kernel boot (**vmlinux-4.9.0-6-4kc-malta**) ~11MB:
```bash
$ wget http://ftp.debian.org/debian/dists/stable/main/installer-mips/current/images/malta/netboot/vmlinux-4.9.0-6-4kc-malta
```
* Optional: Verify downloaded files with [SHA256SUMS](http://ftp.debian.org/debian/dists/stable/main/installer-mips/current/images/SHA256SUMS){: target="_blank" __} by manually comparing the hash values:
```bash
$ shasum -a 256 initrd.gz vmlinux-4.9.0-6-4kc-malta
ea6acc7cc48c7bdb81fd0af2b3546a299e00172539d735001b550b0582b97c0a  initrd.gz
b15610b7aa81d56855627cae7518a396081929ccb28dc1add873322d69f8f08e  vmlinux-4.9.0-6-4kc-malta
```

### Create an QEMU image file
Create an QEMU image file specifying its storage size and filetype to be used as installation media. The table below shows the minimal hardware requirements as per Debian official documentation: [Link](https://www.debian.org/releases/stable/mips/ch03s04.html.en){: target="_blank" __}

Install Type | Minimum (RAM) | Recommended (RAM) | Storage
No Desktop   | 128MB         | 512MB             | 2GB
Desktop      | 256MB         | 1GB               | 10GB

Create an **qcow2** format image with **2G** of storage:

```bash
$ qemu-img create -f qcow2 hda.img 2G
```

### Install Debian MIPS
Before starting make sure all three files (**hda.img**, **vmlinux-4.9.0-6-4kc-malta** and **initrd.gz**) are actually in the current working directory. The installation process is almost identical to the standard x86_64 or i386 architectures.

To start the installation type:
```
$ qemu-system-mips -M malta \
  -m 256 -hda hda.img \
  -kernel vmlinux-4.9.0-6-4kc-malta \
  -initrd initrd.gz \
  -append "console=ttyS0 nokaslr" \
  -nographic
```

By default QEMU enables a NATed network interface for Internet connectivity through the hosts network. This allows the virtual machine to install and update packages.



#### Install SSH server
![alt text]({{page.image}}/mips-ssh-install.png "MIPS SSH Install")
I highly recommend installing a **SSH server** so you can communicate with the host machine for uploading and downloading files whilst in a NATed network. The writer has yet to explore network bridging and other network connectivity. This will probably be the next post.

#### Installation Completed
![alt text]({{page.image}}/mips-installation-done.png "MIPS Installation Complete")

Once you see this screen your installation has completed and it's time to shutdown. Unfortunately, if you hit continue qemu will reboot right back into the installer. Therefore you'd either want to kill process or enter cli shell by selecting **Go Back** > **Go Down** > **Execute Shell** and type command `poweroff` that will shutdown the virtual machine.

### Copy over Kernel initrd.img file
![alt text]({{page.image}}/mips-bootloader.png "MIPS Boot Loader Error")
During the installation stage you'll see this screen warning us that no bootloader has been installed.

Before you can use the freshly installed MIPS image you first need to extract the Kernel initrd.img-[version] file found in the **/boot** partition of the image. We must manually copy it by mounting the image and executing a few commands.

1. Mount the boot partition of the image file:
```bash
sudo modprobe nbd max_part=63
sudo qemu-nbd -c /dev/nbd0 hda.img
sudo mount /dev/nbd0p1 /mnt
```

2. Copy a single file or the entire folder to the current directory:
```bash
cp -r /mnt/boot/initrd.img-4.9.0-6-4kc-malta .  # copy only initrd.img file
cp -r /mnt/boot .                               # copy the entire boot folder
```

3. Unmount the image:
```bash
sudo umount /mnt
sudo qemu-nbd -d /dev/nbd0
```

### Running the QEMU image
Now that all the files have been configured and set up. It's time to officially start the virtual machine. The following set of options can be changed to your liking. You could also make the following into a Bash script.

To start the image type:
```
$ qemu-system-mips -M malta \
  -m 256 -hda hda.img \
  -kernel vmlinux-4.9.0-6-4kc-malta \
  -initrd initrd.img-4.9.0-6-4kc-malta \
  -append "root=/dev/sda1 console=ttyS0 nokaslr" \
  -nographic \
  -device e1000-82545em,netdev=user.0 \
  -netdev user,id=user.0,hostfwd=tcp::5555-:22
```

The last option enables port forwarding on host machine port 5555 to the guest machine on port 22 for ssh communication.

To access the guest machine from Host machine to upload a file:

```bash
$ scp -P 5555 file.txt root@localhost:/tmp
```

Or to connect via ssh:

```bash
$ ssh root@localhost -p 5555
```

#### The result

![alt text]({{page.image}}/mips-screen.png "MIPS-32 Big Endian")

### Thanks
A few other resources that were very helpful:
* [QEMU - Debian Wiki](https://wiki.debian.org/QEMU){: target="_blank" __}
* [Building a Debian Stretch QEMU image for MIPSel - Blah Cats](https://blahcat.github.io/2017/07/14/building-a-debian-stretch-qemu-image-for-mipsel/){: target="_blank" __}
* [Pre-configured Debian Squeeze and Wheezy images](https://people.debian.org/~aurel32/qemu/mips/){: target="_blank" __}
* [OpenWrt in QEMU - OpenWrt Wiki](https://wiki.openwrt.org/doc/howto/qemu){: target="_blank" __}
* [Debian on an emulated MIPS(EL) machine](https://www.aurel32.net/info/debian_mips_qemu.php){: target="_blank" __}
