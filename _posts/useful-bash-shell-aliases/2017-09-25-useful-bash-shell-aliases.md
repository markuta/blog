---
layout: post
title: A Few Handy Bash Shell Aliases
date: 2017-09-25 09:45:47 +07:00
description: A handful of useful bash shell aliases that I've been using for years. They work on just about every default Unix and Linux system, including Mac OS. This post will be updated regularly.
tags: 
    - bash
    - shell
---
### Reload bash_profile
Apply any changes made to ~/.bash_profile with **reload**: `alias reload='source ~/.bash_profile'`

### Network Connections
List all network connections with **nets**: `alias nets='lsof -i'`

### Internet Speed test
Speed test using a 100Mbyte file from OVH Hosting:
```bash
alias speedtest='curl -o /dev/null http://ovh.net/files/100Mio.dat'
```

### WAN IP Address
Show WAN IP address with **myip**: `alias myip='curl ifconfig.co'`

### Web Server Banner
A curl function to grab web server banner information with **headers** followed by a URL:

```bash
headers () { /usr/bin/curl -X GET -I -L -k -A 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36' $@ ; }
```

### Website HTML Source
A curl function to view a web page's html source with **view-source** followed by a URL:

```bash
view-source () { /usr/bin/curl -L -k -A 'Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:25.0) Gecko/20100101 Firefox/25.0' $@ ; }
```

### File Header
View the first few bytes of any file with **filehead** followed by a filename:

```bash
filehead () { /usr/bin/xxd -u -g 1 $@ | /usr/bin/head ;}
```

### List and Directory

```bash
alias cp='cp -iv'
alias mv='mv -iv'
alias mkdir='mkdir -pv'
alias ll='ls -FGlAhp'
alias less='less -FSRXc'
alias CD='cd'
alias cd..='cd ../'
alias ..='cd ../'
alias ...='cd ../../'
alias ....='cd ../../../'
alias .....='cd ../../../../'
alias ~='cd ~'
alias c='clear'
```

### Combined
Just copy and paste within your ~/.bash_profile or ~/.bashrc and run `source`.

```bash
# Reload
alias reload='source ~/.bash_profile'

# Network
alias nets='lsof -i'
alias myip='curl ifconfig.co'
alias speedtest='curl -o /dev/null http://ovh.net/files/100Mio.dat'

# Server Headers
headers () { /usr/bin/curl -X GET -I -L -k -A 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36' $@ ; }

# View Page Source
view-source () { /usr/bin/curl -L -k -A 'Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:25.0) Gecko/20100101 Firefox/25.0' $@ ; }

# File Header
filehead () { /usr/bin/xxd -u -g 1 $@ | /usr/bin/head ;}

# Common
alias cp='cp -iv'
alias mv='mv -iv'
alias mkdir='mkdir -pv'
alias ll='ls -FGlAhp'
alias less='less -FSRXc'
alias CD='cd'
alias cd..='cd ../'
alias ..='cd ../'
alias ...='cd ../../'
alias ....='cd ../../../'
alias .....='cd ../../../../'
alias ~='cd ~'
alias c='clear'
```
