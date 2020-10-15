---
layout: post
title: Alfa AWUS052NH Wireless USB Review
date: 2017-08-16 01:00 +0700
description: Review of Alfa AWUS052NH Dual-Band (2.4GHz and 5GHz) wireless USB adapter. Testing Packet Injection, Deauth attacks, Evil-twin attacks, and more from various different packages available on Kali Linux. This review is should be suited to a Wireless Penetration Tester.
tags: 
    - review
    - wireless
    - aircrack-ng
    - pentest
image: /alfa-awus052nh-review/
---
The **Alfa AWUS052NH** is a high-performance Dual-Band (2.4GHz and 5GHz) wireless USB adapter. It's based on the MediaTek RT3572 chipset that supports IEEE 802.11 **a/b/g/n** standards with up to 300Mbps transfer speeds, it's Alfa's third device in their [802.11abgn USB](https://www.alfa.com.tw/products_list.php?pc=67) product range and has been available since March 2015 which costs around £47 or $60 depending on where you buy it.

[![alt text]({{page.image}}/amazon-button.png "Amazon affiliate link"){: .center-image }](https://amzn.to/2NYlRXB){: target="_blank" __}

This review is geared towards a person with an interest in wireless network security or penetration testing who are contemplating on a purchase, as well as any tech enthusiast. I'll briefly touch on some attacks too. Please don't hesitate to get in contact if I've missed anything.

#### UPDATE: Kali Linux (kernel 4.12.6) has issues with wireless cards.
There seems to be an issue with the recent **4.12.6-1kali1** kernel update which caused really poor performance on a lot of wireless devices. I've re-tested on older kernel **4.11.6-1kali1** and it worked fine, I've written a quick [How-to Downgrade](https://markuta.com/kali-linux-kernel-4-12-wireless-problems/) tutorial for falling back to an older kernel. **Update [2]** Kali Linux has migrated to a new kernel **4.12.13-1kali2** which fixes wireless issues at least for this card.


### Device Specification
Photograph of a setup I previously had, not the current I'll be testing with. From left to right; **Alfa Wireless device**, Raspberry Pi 2, Anker Battery and Macbook pro.

![alt text]({{page.image}}/wireless-setup.jpg "University project setup")

Description | Value
---: | :---
Frequencies | 2.4GHz and 5GHz
Standards | 802.11 a/b/g/n
Chipset | MediaTek RT3572
MIMO | 2x2:2
Wireless Security | WEP, WPA/2, 802.11X & WPS
Operating Modes | IBSS, managed, AP, monitor, WDS, mesh point
Radio Antenna | Two 5dBi detachable antennas with RP-SMA male connectors
Power Output  | Tx-Power **b/g**:30dBm*    **a**:27dBm*
Connector Type | Mini-USB to USB 2.0 Type A

Inside the package:
- Alfa AWUS052NH wireless device (includes a clipper)
- Two 5dBi Dipole Antennas
- Mini-USB to USB 2.0 Type A Y cable (for providing extra power)
- Instruction Manual
- Driver CD

### Setup
I've tried to simulate an attacker's environment with the first three were the main components:
* Laptop running Kali Linux Light `4.11.0-kali1-amd64` on VMware Fusion and Mac OS host machine wireless network card.
* Linksys E2500 Wireless Router (IEEE 802.11 **a/b/g/n**) running Tomato by Shubby firmware for better configuration options and other advanced features.
* Alfa Wireless Network USB Card.
* Mobile devices that have dual-band support.


### Tools & Attacks Tested
The tests were performed on the latest default Kali Linux driver for RT3572 chipsets ([rt2800usb driver](https://wikidevi.com/wiki/Rt2800usb)).

Tested            | 2.4GHz | 5GHz   |  Notes
------------------|:------:|:------:|----------
airodump-ng       | yes    |  yes   | 2.4GHz and 5GHz monitoring and sniffing worked really well.
aireplay-ng       | yes    |  yes*  | deauth attack works on both bands. packet injection shows <br> it works with `-D` option, however more tests are required.
airbase-ng        | yes    |  yes   | worked very well.
wireshark         | yes    |  yes   | no issues analyzing packets while in live monitor mode.
kismet            | yes    |  yes   | no issues.
mdk3              | yes    |  yes*  | deauth attacks works well, may reset access points.
evil-twin         | yes    |  yes   | no issues.
captive portal    | yes    |  yes   | no issues.

#### airodump-ng
Performed very well at passive scanning on both bands, I was able to successfully capture four-way handshakes, channel hop, probe access points and reveal hidden ones too. I've always maintained a strong connection and never had any drop out issues.

Scanning 5GHz frequencies `--band a` or `--channel 36-165` while channel hopping may display (-1), this can be easily resolved by selecting a specific channel rather than channel hopping. The `--ignore-negative-one` doesn't seem to help either.

#### aireplay-ng
Running an aireplay-ng test on **b/g** (2.4GHz) band works really well, supports packet injection and deauth attacks right out of the box, which is expected, results:
```
root@Kali:~# aireplay-ng --test -e XXXXXX -a 20:AA:XX:XX:XX:XX wlan0mon
13:46:18  Waiting for beacon frame (BSSID: 20:AA:XX:XX:XX:XX) on channel 1
13:46:18  Trying broadcast probe requests...
13:46:18  Injection is working!
13:46:20  Found 1 AP

13:46:20  Trying directed probe requests...
13:46:20  20:AA:XX:XX:XX:XX - channel: 1 - 'XXXXXX'
13:46:21  Ping (min/avg/max): 1.458ms/16.169ms/34.363ms Power: -8.07
13:46:21  30/30: 100%
```

~~Unfortunately, aireplay-ng wasn't capable of packet injection on 5GHz frequencies, which is a shame.~~
Running the same test under a specific 5GHz channel (40) with `iwconfig wlan0mon channel 40` beforehand failed, results:
```
root@Kali:~# aireplay-ng --test -e XXXXXX-5G -a 20:AA:XX:XX:XX:XX wlan0mon
13:49:56  Waiting for beacon frame (BSSID: 20:AA:XX:XX:XX:XX) on channel 40
13:50:06  No such BSSID available.
```

Wait what? By giving the `-D` override AP detection, `-a` access point BSSID and `-e` access point ESSID options the test was able to complete, I'm a bit skeptical on this as I've read aircrack-ng 5GHz injection isn't supported properly, anyway here were the results:  

![alt text]({{page.image}}/aireplay-5ghz-test2.png "5GHz Injection Test"){: .center-image }

De-authentication on 5GHz works! Tested on a client connected to the target access point on channel 48 using **802.11an** (40MHz) in range immediately lost connection. Options used `-D` override AP detection, `-a` access point BSSID, `-c` client MAC address and `-e` access point ESSID, results:

![alt text]({{page.image}}/aireplay-deauth-5ghz.png "5GHz Deauth Attack"){: .center-image }

An unusual de-authentication method found when using `--fakeauth`, the key here is to use `-h` option to change the Alfa device MAC address to a client's one which is already associated with a access point, after a few packets the access point sends a deauthentication packet to both us and the real client, results:
```
root@kali:~# aireplay-ng -D --fakeauth 6000 -o 1 -q 10 -a 20:AA:XX:XX:XX:XX \
-h E0:F8:XX:XX:XX:XX -e XXXXXX-5G wlan0mon
```
![alt text]({{page.image}}/aireplay-fakeauth.png "5GHz Fakeauth Attack"){: .center-image }


#### airbase-ng
Create a fake access point for capturing handshakes. Devices with saved profiles will most likely try to connect a better signal network. This attack is specifically useful when it's just the client as only three packets are needed to launch a dictionary attack on the handshake (BSSID, Anonce & Snonce + MIC). The following shows the options used to create the clone listening on channel 44:
```
root@Kali:~# airbase-ng -c 44 -e "XXXXXX-5G" -a 20:AA:XX:XX:XX:XX -W 1 -Z 2 \
-n 484024ace58c73f81692083e987ed9bd33fa5ddf148a6d35181c38f59447e2dc wlan0
```

The `-Z 2` (WPA2 TKIP) option was the best suited in this attack as `-Z 4` (WPA2 CCMP) had difficulties retrieving the 2nd part of the four-way handshake and lead to client having to re-enter their password (not stealthy). One interesting option `-n` sets a specific Anonce value rather than having one randomly generated, this could be a useful aspect for future debugging and testing purposes.

For this attack to be successful it's important the client actually knowns the correct password, otherwise you'll waste time cracking a wrong password.


#### wireshark
Analyzing network packets with wireshark. I was able to view live packets, in this particular case the first two messages of the four-way handshake as well as the custom Anonce, previously setup by airbase-ng, results:

![alt text]({{page.image}}/wireshark-test.png "WPA2-handshake in wireshark")

A few other useful filters:
* Get (1/4) of four-way handshake: `wlan_rsna_eapol.keydes.mic contains 0:0`
* Get (1/4) of four-way handshake TKIP only: `eapol.keydes.key_info == 0x0089`
* Get (2/4) of four-way handshake TKIP only: `eapol.keydes.key_info == 0x0109`
* Get Frames by specific Anonce or Snonce: `wlan_rsna_eapol.keydes.nonce contains 48:40:24`

#### kismet
My second choice when it comes to network sniffers or monitors. It performed very well in identifying networks on both bands. I didn't test any plugins or intrusion detection capabilities but I'm sure there is no reason for them not to work.

#### mdk3
A really noisy method that cripples an access point by connecting thousands of fake client devices which completely destroys the network, in some cases reseting the router. Tests revealed using the following option was difficult to capture handshakes:

`mdk3 wlan0mon a -a 20:AA:XX:XX:XX:XX -m E0:F8:XX:XX:XX:XX`

The de-authentication attack (d option) on the 5GHz 802.11a/n does work. Tested with 802.11a 20MHz + 40MHz, and 802.11an with 20MHz + 40MHz on channels 36, 40, 44 and 48. The de-authentication attack on 2.4GHz band works too.

`mdk3 wlan0mon d -b blacklist.txt -c 40` The file **blacklist.txt** contains a list of BSSID addresses.

#### evil-twin
The evil-twin attack makes use of hostapd, dnsmasq and iptables rules if one plans to phish out credentials. This attack focuses heavily around the hostapd program, for basic functionality a wireless interface card must support AP operating mode, since almost all wireless cards support this mode there were no issues configuring a clone access point to perform our pentesting tasks.

It was also possible to perform a simulated attack against the WPA2 Enterprise 802.11X standard on both bands. The attack itself relies on a patched version of hostapd called [hostapd-wpe](https://github.com/OpenSecurityResearch/hostapd-wpe) which will capture the challenge submitted by a user if they decide to accept an attacker's supplied certificate.

#### captive-portal
Much like evil-twin attack, with additional iptables rules for forwarding traffic and a web server to serve a fake captive-portal page. This attack works really well when clients use hotspot assistant software such as Macs 'Captive Network Assistant' or enable auto-connect features. They'll be asked to provide login credentials in order to proceed for example 'BT-with-FON' wi-fi hotspots found all over Europe.

No special configuration needed as it only requires a device with AP mode.

#### Miscellaneous
Adjusting TX-power output. I was able to achieve 30dBm (bg) and 27dBm (a) specific to each channel by changing country regulatory settings, be sure that is legal under your country's telecommunication laws.

Pairing the wireless device such as a Raspiberry Pi 2 on its own will not work. The RPi2 doesn't provide enough juice to power the ALFA device that's why in the first photograph above it's being powered by a battery pack, it's also the reason they supply a USB Y cable.

Detachable antennas is a real nice feature, even though the writer didn't make use of this it's still an important aspect all external wireless devices should have.

### Summary
The ALFA AWUS052NH wireless USB device, or what I like to referrer it as 'the Orange One' is another great product from Alfa Networks. Its Dual-Band monitoring and sniffing capabilities are a treat to have, more so when considering it's power output compared to other devices on the market. The most surprising aspect about this card was the aireplay-ng test with `-D` option, without which I couldn't even perform a simple de-authentication attack. Overall I can confidently say this device supports passive and active attacks on both bands, as I was able to attack the following; WPA/2 (TKIP), WPA/2 (CCMP), WPA2-Enterprise security standards. The 802.11ac standard wasn't tested as the writer currently doesn't own any supported devices.

It's undoubtably a product worth checking out, especially for pentesters.


#### What's next
Purchase two wireless cards based on the Atheros (AR 9590) chipset which uses the ath9k driver, preferably the [Compex WLE350NX](https://www.amazon.com/dp/B00CA57XZS) mini-pcie card. Atheros chipsets have great reputation when it comes to wireless penetration testing especially packet injection, this particular chipset supports 3x3:3 MIMO and 802.11a/n/b/g both 2.4GHz and 5GHz.

With the addition of a [Laguna GW2387 SBC](http://www.gateworks.com/product/item/laguna-gw2387-network-processor) Single Board Computer or any that supports multiple mini-pice slots working simultaneously. This device would be a sort of like a drop box, a physical device that can be hidden somewhere in range of a target's wireless network and accessed remotely via LTE on the attacker's side.
