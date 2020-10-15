---
layout: post
title: TP-Link Archer T2U Nano for TLS Traffic Interception
date: 2019-11-10 01:00 +0700
description: A guide on how to use a TP-Link Archer T2U Nano AC600 wireless adapter, along with the mitmproxy tool to create a intercepting set-up for inspecting, modifying and monitoring encrypted HTTPS traffic. 
tags: 
 - linux
 - wireless
 - mitmproxy
 - hostapd
image: /assets/img/tp-link-archer-t2u-nano-for-tls-traffic-interception/
site_url: https://markuta.github.io/blog/_posts/tp-link-archer-t2u-nano-for-tls-traffic-interception/
---

### Overview
In this guide we'll be going through the process of configuring an intercepting set-up using mitmproxy and a wireless network, to inspect, modify and monitor encrypted HTTPS traffic. This will allow for a simple way to analyse traffic on mobile handsets and IoT devices, with the only requirements is Wi-Fi support and the ability to install custom certificates.

> **mitmproxy** is your swiss-army knife for debugging, testing, privacy measurements, and penetration testing. It can be used to intercept, inspect, modify and replay web traffic such as HTTP/1, HTTP/2, WebSockets, or any other SSL/TLS-protected protocols. [read more](https://mitmproxy.org/){: target="_blank" __}

A simple illustration of mitmproxy running in transparent mode under Debian Buster:

![alt text]({{page.image}}/mitmproxy-example.png "mitmproxy example"){: .center-image }

The communication process:
```
iPhone <==> Wi-Fi TAP  <==> mitmproxy <== (IP Forwarding) <==> Internet
```

### Requirements

#### Software
There are several software packages required for this set-up. All of which can be downloaded through `apt-get` on most Linux based systems. As mentioned previously I'll be using Debian Stretch, but the guide can be applied to other systems.

List of required packages:
* `mitmpoxy` – a HTTP traffic interception tool. 
* `dnsmasq` - a lightweight DNS forwarder and DHCP service. 
* `hostapd` – a utility used for creating the wireless access point. 
* `tcpdump` – an all-round great packet capture utility.

To install all of the above: 

```bash
sudo apt update
sudo apt install mitmproxy hostapd dnsmasq tcpdump
```

#### Hardware
And for hardware, the only requirement is having two network interface cards, one for providing Internet connectivity and the other for broadcasting the wireless access point. I'll be using one I bought from [Amazon](https://www.amazon.co.uk/gp/search?ie=UTF8&tag=markuta-21&linkCode=ur2&linkId=50c49b93260f03f980557ecae1f77abd&camp=1634&creative=6738&index=computers&keywords=TP-Link Archer T2U AC600 nano){: target="_blank" __}  `TP-Link Archer T2U Nano AC600` which is cheap and supports 2.4GHz and 5GHz natively on Linux systems, and my MacBook Pro's built-in wireless card.

![alt text]({{page.image}}/tp-link-product.jpg "mitmproxy example"){: .center-image }


#### Drivers
The TP-Link Archer T2U Nano AC600 WiFi adapter requires additional effort while installing on Linux. The only driver that worked was `realtek-rtl88xxau-dkms` version **5.2.20.2~20190617** available on [here](https://gitlab.com/kalilinux/packages/realtek-rtl88xxau-dkms){: target="_blank" __}. 

```bash
git clone https://gitlab.com/kalilinux/packages/realtek-rtl88xxau-dkms
cd realtek-rtl88xxau-dkms
sudo ./dkms-install.sh 
```
To check if the driver installed properly, plug in your USB adapter and ensure pass-through is enabled within the virtualization software. In my case, VMware Fusion (Menu > Virtual Machine > USB & Bluetooth).

![alt text]({{page.image}}/mitmproxy-vmware-interface.png "mitmproxy vmware interface"){: .center-image }

Run the following inside the virtual machine `ip a`:

![alt text]({{page.image}}/mitmproxy-wireless-interface.png "mitmproxy wireless interface"){: .center-image }

There should be two network interface cards, one called `ens33` which will provide access to the Internet, and the other interface we just created called `wlxd037XXXXXXXX` will be configured as our wireless access point to use with mitmproxy.


### Configure

#### Networking
Ensure the newly created network interface has a static IP address and a gateway address. This will point to the Dnsmasq service in the step next. Use the following network configuration and replace network interfaces to suite your own. A reboot may be required.

Network configuration file: `/etc/network/interfaces`

```conf
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens33
iface ens33 inet dhcp

# The secondary network interface
allow-hotplug wlxd037XXXXXXXX
iface wlxd037XXXXXXXX inet static
        address 10.0.0.254/24
        netmask 255.255.255.0
        gateway 10.0.0.254
```

#### IP Forwarding and IPtables
Start by enabling IP forwarding and add masquerade rules with IPtables to route and redirect specific network traffic to mitmproxy. This will be persistent.

Edit the main system config: `/etc/sysctl.conf` and uncomment this line:
```conf
net.ipv4.ip_forward=1
```

Apply the following rules to IPTables (replace to suit your wireless interface):
```bash
sudo iptables -t nat -A PREROUTING -i wlxd037XXXXXXXX -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -i wlxd037XXXXXXXX -p tcp --dport 443 -j REDIRECT --to-port 8080
```

#### Dnsmasq
To make devices have their own IP addresses assigned automatically, a DHCP server is required. Dnsmasq is a lightweight DNS forwarder and DHCP server suitable for small networks. This makes sense otherwise we would need to manually assign addresses.

Create and modify the configuration file: `/etc/dnsmasq.conf` and replace it with:
```
# Wireless Interception Interface
interface=wlxd037XXXXXXXX

# DHCP server and range for assigning IP addresses
dhcp-range=10.0.0.1,10.0.0.100,96h

# Broadcast gateway and DNS server information
dhcp-option=option:router,10.0.0.254
dhcp-option=option:dns-server,10.0.0.254
```

#### Hostapd
And finally, create the wireless network with Hostapd. This wireless network will operate under the 2.4GHz frequency (channel 7) and will be able to forward traffic onto the primary interface. 

Create a new configuration file: `/etc/hostapd/hostapd.conf`
```conf
interface=wlxd037XXXXXXXX
driver=nl80211
ssid=TAP
hw_mode=g
channel=7
ht_capab=[HT40][SHORT-GI-20]
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=gimme_the_loot
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP TKIP
rsn_pairwise=CCMP
```

To use the 5GHz band with TP-Link adapter the Hostapd configuration file needs to be changed and driver needs to be loaded with: 
```bash
modprobe -r 88XXau && modprobe 88XXau rtw_vht_enable=2
```

#### mitmproxy
Saving TLS master keys to a file while running mitmproxy. With this file Wireshark is able to decrypt TLS traffic. See the [Wireshark wiki](https://wiki.wireshark.org/TLS?action=show&redirect=SSL#Using_the_.28Pre.29-Master-Secret){: target="_blank" __} for more information. Modify your `.bashrc` file and export the environment variable:
```bash
export SSLKEYLOGFILE="$HOME/sslkeylogfile.txt"
```

#### tcpdump
When you want to capture all network traffic from an interface and you do not want to intercept or modify the flow. You need to flush the previous IPTables rules and apply a new rule.

To flush all IPTable rules within the nat table:
```bash
sudo iptables -t nat -F
```

And add a new rule to masquerade traffic to the Internet interface:
```bash
sudo iptables -t nat -A  POSTROUTING -o ens33 -j MASQUERADE
```

After that you can run a tcpdump packet capture:
```bash
sudo tcpdump -i <interface> -s 65535 -w <file>
```


### Launching

First, start the DNSmasq server: 
* `sudo systemctl start dnsmasq`

Next, start the Hostapd service:
* `sudo systemctl start hostapd`

Finally, start the mitmproxy for example in transparent mode:
* `mitmproxy --mode transparent --showhost`


#### iPhone
Here is an example of monitoring traffic using an iPhone device. First, connect to the wireless network created by Hostapd as you would to a regular Wi-Fi network.

![alt text]({{page.image}}/IMG_5554.png "mitmproxy wireless network"){: .center-image }

Then, open a browser and navigate to `http://mitm.it` (make sure mitmproxy is running). Download the certificate for your device type and hit allow.

![alt text]({{page.image}}/IMG_5551.png "mitmproxy certificate"){: .center-image }

Next, go to **Settings > General > Profile > mitmproxy > Install Profile**. When installed, go back to the main Settings again **General > About > Certificate Trust Settings** and enable full trust for root certificate.

![alt text]({{page.image}}/IMG_5552.png "mitmproxy add profile"){: .center-image }



### Fixes
There appears to be an issue with mitmproxy packages on Debian (Stretch and Buster). When visiting the `http://mitm.it/` to download certificates, the page does not display the links properly. Leaving the user to manually browse site on devices.

* `http://mitm.it/cert/pem` – Android & iOS
* `http://mitm.it/cert/p12` – Windows


Or find and edit the file `/usr/lib/python3/dist-packages/mitmproxy/addons/onboardingapp/templates/layouts.html` and replace:

```html
<link href="/static/bootstrap.min.css" rel="stylesheet">
<link href="/static/font-awesome.min.css" rel="stylesheet">
```
With:
```html
<link href="//stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" rel="stylesheet">
<link href="//stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet">
```

### Resources
A list of useful resources:
* [How to use Transparent mode in VMs](https://docs.mitmproxy.org/stable/howto-transparent-vms/)
* [Running a Man-in-the-middle proxy on a Raspberry pi 3](https://www.dinofizzotti.com/blog/2019-01-09-running-a-man-in-the-middle-proxy-on-a-raspberry-pi-3/)
* [Raspberry Pi as Wireless Access Point](https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md)
* [Four ways to Bypass Certificate Pinning on Android](https://blog.netspi.com/four-ways-bypass-android-ssl-verification-certificate-pinning/)
* [Bypass SSL Pinning Android](https://v0x.nl/articles/bypass-ssl-pinning-android/)


