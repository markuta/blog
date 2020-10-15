---
layout: post
title: Live Memory Acquisition on Linux Systems
date: 2018-08-26 01:00 +0700
description: How to acquire a live memory image dump from a Linux system using the LiME Kernel Module. Perform memory analysis using Volatility with a custom Linux profile.
tags: 
    - forensics
    - memory
    - lime
    - volatility
    - linux
image: /live-memory-acquisition-on-linux-systems/
---

In this blog post I'll be demonstrating a process of obtaining or acquiring a memory image from a running Linux system. The tool of choice LiME (Linux Memory Extractor) and is available on [Github](https://github.com/504ensicsLabs/LiME){: target="_blank" __}.

After a forensic image has been acquired we will use [Volatility](https://github.com/volatilityfoundation/volatility){: target="_blank" __} with a custom Linux profile for the analysis, to keep things simple I've used the latest Debian Stretch kernel version `4.9.0-8-amd64` as the target system so it's easily repeatable.

![alt text]({{page.image}}/volatility.png "Volatility")


### Building LiME Kernel Module

> LiME (formerly DMD) is a Loadable Kernel Module (LKM), which allows the acquisition of volatile memory from Linux and Linux-based devices, such as those powered by Android. [read more](https://github.com/504ensicsLabs/LiME)

To use the Kernel module it must be built for that specific Kernel version, otherwise insmod will not be able to load it. There are also times when targets may use non-standard Kernels i.e. Grsecurity or completely custom ones.

Ideally the following should be done on a forensics workstation. But there are times when it may be necessary to compile and load the module directly on a target system, an example would be when a custom Linux Kernel is present.

##### Prerequisites
```bash
sudo apt install linux-headers-4.9.0-8-amd64
sudo apt install build-essential
```

##### Download LiME
```
git clone https://github.com/504ensicsLabs/LiME
```

##### Compile
```bash
cd LiME/src/
make
```

You'll notice a file being created named `lime-4.9.0-8-amd64.ko` this is our LKM.

### Memory Acquisition
With LiME you have the option to either write to disk or transfer over the network. The latter may pose issues with firewalls or high network usage environments. Nevertheless, the option is there, for our purposes we will be writing to a disk. In a real world scenario you'd be writing to some form of external media.

#### Write to disk
The following uses **insmod** to load our compiled Loadable Kernel Module. The options `format=lime` and `timeout=0` are important for Volatility. Testing revealed there are a few issues with type `raw`.

```bash
sudo insmod lime-4.9.0-8-amd64.ko "path=/media/external/dump.mem format=lime timeout=0"
```

#### Write over the network
Similar to the above we start a listening session on a port 4444 using `tcp:port`:

```bash
sudo insmod lime-4.9.0-8-amd64.ko "path=tcp:4444 format=lime timeout=0"
```

On a remote host workstation we can use **netcat** to establish a connection and download the image:
```bash
 nc 10.10.1.10 4444 > dump.mem
```

#### Cleaning up
When complete to unload the Kernel Module simply type: `sudo rmmod lime`


###	Building Linux Volatility Profile
Building a Linux profile for Volatility requires a bit more effort. There are times when it may not work correctly. Many people have experienced issues with Linux Kernel versions **4.8+** due to the way Kernel address space layout randomization (KASLR) works.

As mentioned above in this blog post we'll be building a Linux profile for Debian Stretch system with the Kernel Version: `4.9.0-8-amd64 (2018-08-21)` which is confirmed working with Volatility.

##### Download Volatility
```bash
git clone https://github.com/volatilityfoundation/volatility
```

##### Install Dependencies
```bash
sudo apt install dwarfdump pcregrep libpcre++-dev python-dev python-pip
```
##### Install Python Modules
```bash
pip install pycrypto Distorm3 OpenPyxl ujson
```

#### Building a Profile
Navigate to `volatility/tools/linux` and type the following:
```bash
sudo make -C /lib/modules/$(uname -r)/build CONFIG_DEBUG_INFO=y M=$PWD modules
dwarfdump -di ./module.o > module.dwarf
sudo zip Debian4908.zip module.dwarf /boot/System.map-$(uname -r)
```

Move or copy the created zip file to the following directory within Volatility:
```bash
cp Debian4908.zip ../../plugins/overlays/linux/
```

Now when running `--info` we should see our newly created Linux Profile(s) **LinuxDebian4908x64** as available. The archive we created will be prepended with _Linux_ and appended with _x64_ dependent on the architecture type.

![alt text]({{page.image}}/volatility-linux-profile.png "Newly created Linux Profiles")

### Memory Analysis with Volatility
Now comes the fun part. Once everything is set up correctly and we've acquired a forensic image using LiME. We can start our analysis with Volatility.

An example command using options `-f` memory file, `--profile` profile name and `linux_banner` plugin would look something like this:

```bash
python vol.py -f debian-latest.lime --profile=LinuxDebian4908x64 linux_banner
```

#### Tested Plugins
The following is a list of working plugins under our profile.

```
linux_arp                  - Print the ARP table
linux_aslr_shift           - Automatically detect the Linux ASLR shift
linux_banner               - Prints the Linux banner information
linux_bash                 - Recover bash history from bash process memory
linux_bash_env             - Recover a process' dynamic environment variables
linux_bash_hash            - Recover bash hash table from bash process memory
linux_check_fop            - Check file operation structures for rootkit modifications
linux_check_idt            - Checks if the IDT has been altered
linux_check_modules        - Compares module list to sysfs info, if available
linux_check_tty            - Checks tty devices for hooks
linux_cpuinfo              - Prints info about each active processor
linux_dmesg                - Gather dmesg buffer
linux_dump_map             - Writes selected memory mappings to disk
linux_dynamic_env          - Recover a process' dynamic environment variables
linux_elfs                 - Find ELF binaries in process mappings
linux_enumerate_files      - Lists files referenced by the filesystem cache
linux_find_file            - Lists and recovers files from memory
linux_getcwd               - Lists current working directory of each process
linux_hidden_modules       - Carves memory to find hidden kernel modules
linux_ifconfig             - Gathers active interfaces
linux_info_regs            - It's like 'info registers' in GDB. It prints out all the
linux_iomem                - Provides output similar to /proc/iomem
linux_kaslr_shift          - Automatically detect KASLR physical/virtual shifts and alternate DTBs
linux_kernel_opened_files  - Lists files that are opened from within the kernel
linux_keyboard_notifiers   - Parses the keyboard notifier call chain
linux_ldrmodules           - Compares the output of proc maps with the list of libraries from libdl
linux_library_list         - Lists libraries loaded into a process
linux_librarydump          - Dumps shared libraries in process memory to disk
linux_list_raw             - List applications with promiscuous sockets
linux_lsmod                - Gather loaded kernel modules
linux_lsof                 - Lists file descriptors and their path
linux_malfind              - Looks for suspicious process mappings
linux_memmap               - Dumps the memory map for linux tasks
linux_moddump              - Extract loaded kernel modules
linux_mount                - Gather mounted fs/devices
linux_netfilter            - Lists Netfilter hooks
linux_netscan              - Carves for network connection structures
linux_netstat              - Lists open sockets
linux_pidhashtable         - Enumerates processes through the PID hash table
linux_pkt_queues           - Writes per-process packet queues out to disk
linux_plthook              - Scan ELF binaries' PLT for hooks to non-NEEDED images
linux_proc_maps            - Gathers process memory maps
linux_proc_maps_rb         - Gathers process maps for linux through the mappings red-black tree
linux_procdump             - Dumps a process's executable image to disk
linux_process_hollow       - Checks for signs of process hollowing
linux_psaux                - Gathers processes along with full command line and start time
linux_psenv                - Gathers processes along with their static environment variables
linux_pslist               - Gather active tasks by walking the task_struct->task list
linux_psscan               - Scan physical memory for processes
linux_pstree               - Shows the parent/child relationship between processes
linux_strings              - Match physical offsets to virtual addresses (may take a while, VERY verbose)
linux_threads              - Prints threads of processes
linux_tmpfs                - Recovers tmpfs filesystems from memory
linux_volshell             - Shell in the memory image
```

#### Example Output

![alt text]({{page.image}}/volatility-example.png "Volatility example")


### Cavets
There may be issues with targets with unique kernel versions or those that utilize  additional SDK. Not to mention further issues with KASLR on newer Linux Kernels.

### Reading Material

- [Linux Memory Extractor Documentation](https://github.com/504ensicsLabs/LiME/tree/master/doc){: target="_blank" __}
- [Volatility Wiki](https://github.com/volatilityfoundation/volatility/wiki/Installation){: target="_blank" __}
- [KASLR and Volatility](https://bneuburg.github.io/volatility/kaslr/2017/04/26/KASLR1.html){: target="_blank" __}
