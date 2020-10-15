---
layout: post
title: What I Use for Password Management
date: 2017-09-25 01:00 +0700
description: What I personally use for Password management and a comparison against cracking encrypted databases files with other password managers.
tags: 
    - passwords
    - management
image: /my-password-management/
---
Lets face it remembering passwords for dozens of sites is a pain which is why some people re-use their password or change it very slightly to avoid the hassle. If you're one of those who would rather generate random complex passwords for each site, the question on how those are stored will arise. Storing passwords in plaintext text file on your desktop is a big **no no**.

Password Managers are great, when they're implemented correctly. I'm personally not a fan of cloud based password managers due to privacy concerns and not knowing how exactly **all** my passwords are actually stored and handled. When my own machine gets compromised, that'll be down to me not properly configuring or following best security practices, but when its out of my control it's and knowing that I took every necessary step to ensure my data is safe, and yet still, somehow all "keys to my castle" get comprised or leak information. I'd prefer to not risk any chances and can do without Cloud services or proprietary software.

A list of Password Management services that have been compromised or do stuff that'll make you worry:
* OneLogin - Breached [^1]
* 1Password - Leaking Data [^2]
* LastPass - Security Breached 2015 [^3], Security Issue 2016 [^4]

List of other password managers: [https://en.wikipedia.org/wiki/List_of_password_managers](https://en.wikipedia.org/wiki/List_of_password_managers)

### KeepassXC
As stated on their website, KeepassXC is a community fork from KeePassX, a native cross-platform port of KeePass Password Safe repository. It is developed in C++ and runs natively on all three supported platforms (Linux, Windows and Mac OS). The interface is simple and straight forward to use, I personally think icons may be improved.

A really nice Setup Guide can be found: [https://sts10.github.io/2017/06/27/keepassxc-setup-guide.html](https://sts10.github.io/2017/06/27/keepassxc-setup-guide.html)

#### Locked Window
![alt text]({{page.image}}/keepassxc-locked.png "KeePassXC Locked Database Window")

#### Unlocked Window
![alt text]({{page.image}}/keepassxc-unlocked.png "KeePassXC Unloocked Database Window")

#### Password Generator
![alt text]({{page.image}}/keepassxc-gen.png "KeePassXC Password Generator Window")


### What works for me
By far the most relevant aspects to me are:

1. Your wallet works offline and requires no Internet connection.
2. I'm in full control of my password manager.
3. Cross-platform support; Linux, Unix, Mac OS and Windows.
4. Open Source - code review and option to implement custom features :)
5. Support for additional authentication methods (Yubi-key, Keyfile)


### Cracking the Encrypted Database
Like with any other security application, I'm always curious on how it handle certain situations, one of those things is the way in which KeePassXC encrypts the main database file.

> What if an adversary has a copy of your encrypted database, how difficult would be cracking it?

Well, that depends on your master key of course. Not to mention any additional protection (key file or Challenge response) you've got set up.

Based on Hashcat benchmark test data provided by [Jeremi Gosney](https://gist.github.com/epixoip/ace60d09981be09544fdd35005051505) which was running on a dedicated password crunching monster the [Sagitta Brutalis](https://sagitta.pw/hardware/gpu-compute-nodes/brutalis/). A system featuring x8 **Nvidia GTX 1080 Ti FE** GPUs and x2 **8-core Intel E5-2600 v4 Xeons** for only $21,169.00 - base price.

The table below shows cracking performance against three other password managers using only the master password and their algorithms which are currently supported by `Hashcat 3.5.0-22-gef6467b`. The speed result is the combined power of the eight GPU cards and not of an individual card.

| Hashtype    | Speed (kH/s)|
|:-----------:|:-----------:|
| 1Password, cloudkeychain  | 137.7 kH/s|
| **KeePass 1 (AES/Twofish) and KeePass 2 (AES)** | **1733.4 kH/s** |
| Password Safe v2 | 4024.7 kH/s |
| Password Safe v3 | 15378.8 kH/s |
| LastPass + LastPass sniffed | 29242.3 kH/s |
| 1Password, agilekeychain | 40477.9 kH/s |

1Password (cloudkeychain) performed surprisingly really well, resulting in roughly about 10 times more computationally difficult to process than KeePass (AES). Keep in mind, KeePass and many others allow users to include an additional authentication method such as Key File and Challenge-Response, without which no adversary can crack.

[^1]: [Password Manager OneLogin hit by data breach; BBC](http://www.bbc.co.uk/news/technology-40118699)
[^2]: [1Password Leaks Your Data; myers.io](https://myers.io/2015/10/22/1password-leaks-your-data/)
[^3]: [Lastpass Breached; nakedsecurity.sophos.com](https://nakedsecurity.sophos.com/2015/06/16/bad-news-lastpass-breached-good-news-you-should-be-ok/)
[^4]: [Lastpass Vulnerability (Fixed); labs.detectify.com](https://labs.detectify.com/2016/07/27/how-i-made-lastpass-give-me-all-your-passwords/)
