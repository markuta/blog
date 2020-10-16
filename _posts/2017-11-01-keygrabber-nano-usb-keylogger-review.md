---
layout: post
title: KeyGrabber Nano USB Keylogger Review
date: 2017-11-01 09:45:47 +07:00
description: Review of the KeyGrabber Nano USB Keylogger. An advanced tiny hardware USB keylogger that supports multinational keyboard layouts and languages. This review is of the non Wi-Fi edition.
tags: 
    - passwords
    - hardware
    - keylogger
image: /assets/img/keygrabber-nano-usb-keylogger-review
---
Whilst studying at University I became interested in hardware keyloggers, and so decided to purchase one for a research paper. This device is specifically for wired USB keyboards (non Wi-Fi or Bluetooth) which records every single key stroke typed, without the need for drivers or worrying about any security product. I thought why not write a review and share some interesting findings.

I DO NOT condone or encourage the use of any such devices for ILLEGAL purposes.

![alt text]({{page.image}}/connected-keyboard.png "KeyGrabber Nano connected to a keyboard")

For those of you who are thinking - Why would you connect it to a laptop? - I currently do not have a tower computer, so I decided to use what I had available, my laptop and a Logitech USB keyboard. No disassembly or reverse engineering was attempted, at this time.

### The Device
The **[KeyGrabber Nano keylogger](https://goo.gl/ERLCsW){: target="_blank" __}** device is the smallest USB hardware keylogger available on the market. The Nano, as the name suggests, has the dimensions of just: 35mm (L) x 20mm (H) x 12mm (W). There are two versions; one with Wi-Fi and the other without Wi-Fi. This review is based on the (non Wi-Fi edition) which is currently available from US for $55.99 or EU for €51.99. Other locations may vary in prices, due to shipping and packages. The company which sells it are called 'KeeLog' in (EU) and 'Aqua Electronics Inc' in (US).

Most typical hardware keyloggers are somewhat noticeable and rather bulky, even though they're often out of sight and fitted to back of systems. With a quick glance those can be seen from a mile away.

Here's the device compared to a British Five Pence Coin:

![alt text]({{page.image}}/hw-keylogger.png "KeyGrabber Nano and 5 Pence")


#### Features
* Compact and Discrete
* Memory capacity (16 MB) flash file system
* Compatible with all USB keyboards (including Linux & Mac)
* Transparent to computer operation, undetectable for security scanners
* No software or drivers required and operating system independent
* Memory protected with strong 128-bit encryption
* Quick and easy national keyboard layout support

### Support
The device supports most wired USB type keywords including backlit and non-backlit models, power ratings 4.5V – 5.5V DC (from USB port). As this is a _hardware_ keylogger it works on all operating systems; Windows, Mac OS, Linux with no issues. I've tested USB versions 2.0 and 3.0, which works perfectly. Here's an example of my Logitech K740 illuminated keyboard (Power rating 5V == 300mA) connected to a Debian system, both with and without the device.

Output of **dmesg**:
```
With device on USB 3.0 port
...
[  144.384517] usb 1-1: new full-speed USB device number 6 using xhci_hcd
[  144.531879] usb 1-1: New USB device found, idVendor=046d, idProduct=c318
[  144.531893] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  144.531903] usb 1-1: Product: Logitech Illuminated Keyboard
[  144.531912] usb 1-1: Manufacturer: Logitech
[  146.694372] input: Logitech Logitech Illuminated Keyboard as /devices/pci0000:00/0000:00:14.0/usb1/1-1/1-1:1.0/0003:046D:C318.0001/input/input12
[  146.752576] hid-generic 0003:046D:C318.0001: input,hidraw0: USB HID v1.11 Keyboard [Logitech Logitech Illuminated Keyboard] on usb-0000:00:14.0-1/input0
[  146.753391] input: Logitech Logitech Illuminated Keyboard as /devices/pci0000:00/0000:00:14.0/usb1/1-1/1-1:1.1/0003:046D:C318.0002/input/input13
[  146.812373] hid-generic 0003:046D:C318.0002: input,hiddev0,hidraw1: USB HID v1.11 Device [Logitech Logitech Illuminated Keyboard] on usb-0000:00:14.0-1/input1
...

Without device on USB 3.0 port
...
[  392.120268] usb 1-1: new full-speed USB device number 8 using xhci_hcd
[  392.266333] usb 1-1: New USB device found, idVendor=046d, idProduct=c318
[  392.266347] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  392.266357] usb 1-1: Product: Logitech Illuminated Keyboard
[  392.266365] usb 1-1: Manufacturer: Logitech
[  392.274911] input: Logitech Logitech Illuminated Keyboard as /devices/pci0000:00/0000:00:14.0/usb1/1-1/1-1:1.0/0003:046D:C318.0003/input/input14
[  392.334189] hid-generic 0003:046D:C318.0003: input,hidraw0: USB HID v1.11 Keyboard [Logitech Logitech Illuminated Keyboard] on usb-0000:00:14.0-1/input0
[  392.339795] input: Logitech Logitech Illuminated Keyboard as /devices/pci0000:00/0000:00:14.0/usb1/1-1/1-1:1.1/0003:046D:C318.0004/input/input15
[  392.397846] hid-generic 0003:046D:C318.0004: input,hiddev0,hidraw1: USB HID v1.11 Device [Logitech Logitech Illuminated Keyboard] on usb-0000:00:14.0-1/input1
...
```

As you can tell it's completely transparent to the OS.

#### Different Keyboard Layouts
There are a total of **48** different national keyboard layouts supported. This is an important aspect if you're considering using the device in different countries. The layouts ensure each key stroke is processed and stored correctly. This option is not enabled by default, but can be easily changed by updating the configuration file to match a chosen keyboard layout.

Below is a list of all supported languages and layouts:

![alt text]({{page.image}}/keyboard-layouts.png "Keyboard Layout List")

Each folder contains a filename called **layout.usb** which is placed onto the root of the USB. In addition, the configuration file needs to be updated with the setting: `DisableLayout=No`. The full and updated list can be downloaded from [here](https://www.keelog.com/download.html#layout){: target="_blank" __}.

### Configuration
The device can be configured through the **config.txt** file located on the root of the USB. Data Storage mode must be activated to either view or edit the files. The most relevant parameters are listed below:

**Parameter** | **Values** | **Example** | **Description**
Password | 3-character password (default KBS) | Password=SVL | Three-character key combination for activating Flash Drive mode.
Encryption | Yes, No (default) | Encryption=No | Flash drive AES encryption
LogSpecialKeys | None, Medium (default), Full | LogSpecialKeys=Full | Special key logging level.
DisableLogging | Yes, No (default) | DisableLogging=Yes | Keystroke logging disable flag.
DisableLayout | Yes, No (default) | DisableLayout=Yes | National layout disable flag.

### Data Storage Mode
The storage mode is activated when a 3-key combination is in typed simultaneously through a connected USB keyboard. Once pressed a new **16MB** storage device will pop-up entitled "KEYGRABBER" and the keyboard will stop functioning, unless it's reconnected. The default secret key combination is (**K** + **B** + **S**). This can be changed to any 3 characters given their unique from a typical keyboard layout by editing the `Password` parameter within the configuration file.

![alt text]({{page.image}}/device-storage.png "Device Storage Model Files")

Example output of **log.txt** (LogSpecialKeys set to Medium):
```
[Pwr]example.com[Ent]
news.ycombinator.com[Ent]
[Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck][Bck]google.com[Ent]
[Alt][Tab][Win][Tab][Win][Tab][Win][Tab][Win][Tab][Win][Tab][Win][Tab]example[Ent]
sb[Pwr][Cap]g[Cap]reat for getting [Cap]bios p[Cap]asswords ()[Cap]*[Ent]
[Ent]p$ssw0rd123%`[Ent]
[Ent][Tab]qwertyuiop[][Ent]
[Cap]asdfghjkl;'#[Ent]
\zxcvbnm,./[Ent]
[F6][F5][Esc][F11][F11]sb
```

### Encryption
In addition to a secret key, there's also an option to enable 128-bit AES encryption to "protect" against device tampering. However, it is let down by the password combination as mentioned below. This option is disabled by default but can be enabled by editing the **config.txt** file and setting `Encryption=Yes`. Doing so will also re-format and purge logged data.

### Drawbacks
Reading the log file may become confusing for none technical users, due to the fact the device records raw keyboard entries, especially when the `LogSpecialKeys=Full` option is selected. The device does come with software called 'KL Tools' which assists users in configuring and viewing recorded data by using an intuitive GUI. However, others may prefer writing their own phrasing scripts.

Encryption option and password key. The biggest drawback is the 3-key secret password. For example, assume a password uses [a-z] characters, that would leave a keyspace of only 26 x 25 x 24 = 15,600. The keys also need to be entered simultaneously. So, if the password is KBS, any permutation of that will activate storage mode e.g. (SBK or BSK) lowering the keyspace even further. All it would take is a modified HID device connected to the keylogger to perform a simple dictionary attack, regardless whether encryption option is enabled or not.

While a USB keyboard is attached, part of the USB male connector is exposed (as shown in the first photograph). It would've of been much cleaner if it had a completely 'flush' finish.

### Summary
The main advantages are its size and support for multinational keyboard layouts. I really do like the unique storage mode activation by way of a 3-key password combination. However, it would've been a much better solution to allow a longer password entered in sequence, that would recognize by pattern and then enter into storage mode. Although, I don't think there's much information to be obtained
for any investigative purposes, possibly file Metadata. Regardless it's an issue a user should be aware of if a device were to be found.

The KeyGrabber Nano is no doubt an interesting device that I've had much fun playing around with. If you are seeking a small covert USB keylogger device this would be my first choice. I really would've liked to try out the Wi-Fi version, if only the cost wasn't so high. In a future post, I may show a brute-force attack against the device using a Rubber Ducky USB (HID).
