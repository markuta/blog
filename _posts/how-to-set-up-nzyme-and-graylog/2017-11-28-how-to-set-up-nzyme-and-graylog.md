---
layout: post
title: How to Set up Nzyme and Graylog
date: 2017-11-28 09:45:47 +07:00
description: A how to guide on setting up Nzyme and Graylog v2.3.2 using a Docker compose file. This tutorial can be applied to any platform running Docker software. Includes examples and configuration files.
tags: 
  - wireless
  - howto
  - nzyme
  - graylog
image: /how-to-set-up-nzyme-and-graylog/
---

This guide shows you how to quickly set up and get started with Nzyme and Graylog version 2.3.2 using Docker. In this tutorial I'm using Mac OS but Docker can be install on any platform. For testing purposes, I'd recommend using Kali Linux or any Debian based distro on where Nzyme sensor is installed.

### What is it?
Nzyme collects 802.11 management frames directly from the air and sends them to a Graylog (Open Source log management) setup for WiFi IDS, monitoring, and incident response. It only needs a JVM and a WiFi adapter that supports monitor mode. Way more information is available on [wtf.horse](https://wtf.horse/2017/10/02/introducing-nzyme-wifi-802-11-frame-recording-and-forensics/) and [GitHub](https://github.com/lennartkoopmann/nzyme).

In a real world scenario users would want to use an additional WWAN or LTE network interface, for when a LAN isn't reachable or unavailable. Below demonstrates a simple network diagram that has two devices running nyzme, both have a number of network interfaces for collecting wireless traffic and communicating back to the centralised server. 

This is how I would picture a simple network architecture: 


![alt text]({{page.image}}/nzyme-network-diagram.png "A Simple Network Diagram")

### Overview
A review of necessary steps. This guide can be applied to any Operating System running Docker software.

* Installing Docker and using a **docker-compose.yml** configuration file that contains defined services (Graylog, MongoDB, ElasticSearch), networks and volumes. 
* Setting up a new GELF input through the Graylog web interface.
* Setting up and deploying a Nzyme sensor.


### Docker
Docker provides containers and images that make it really easy to install and configure Graylog server. Mongo Database and ElasticSearch are both required by Graylog. Rather than installing by hand we could define them in our **docker-compose.yml** file. It's also important to use the right software version, as Graylog is picky on what version is used. Without specifying the version, it'll use the latest which in some cases isn't a good idea.

Download and install docker: [https://www.docker.com](https://www.docker.com/)

A quick note on software versions:
* MongoDB 3.0.15
* ElasticSearch 5.5.2
* Graylog 2.3.2

Network ports:
* 514 - Syslog UDP/TCP
* 9000 - Graylog Web Interface
* 12201 - GELF TCP/UDP

#### Docker-Compose file
```yaml
version: '2'
services:
  # MongoDB: https://hub.docker.com/_/mongo/
  mongo:
    image: mongo:3
    # Persistent Logs
    volumes:
      - mongo_data:/data/db
  # Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/5.5/docker.html
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.5.2
    # Persistent Logs
    volumes:
      - es_data:/usr/share/elasticsearch/data
    environment:
      - http.host=0.0.0.0
      # Disable X-Pack security: https://www.elastic.co/guide/en/elasticsearch/reference/5.5/security-settings.html#general-security-settings
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
  # Graylog: https://hub.docker.com/r/graylog/graylog/
  graylog:
    image: graylog/graylog:2.3.2-1
    # Persistent Logs
    volumes:
      - graylog_journal:/usr/share/graylog/data/journal
    environment:
      # CHANGE ME!
      - GRAYLOG_PASSWORD_SECRET=somepasswordpepper
      # Password: admin
      - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
      - GRAYLOG_WEB_ENDPOINT_URI=http://127.0.0.1:9000/api
    links:
      - mongo
      - elasticsearch
    ports:
      # Graylog web interface and REST API
      - 9000:9000
      # Syslog TCP
      - 514:514
      # Syslog UDP
      - 514:514/udp
      # GELF TCP
      - 12201:12201
      # GELF UDP
      - 12201:12201/udp
# Volumes for persisting data, see https://docs.docker.com/engine/admin/volumes/volumes/
volumes:
  mongo_data:
    driver: local
  es_data:
    driver: local
  graylog_journal:
    driver: local

```

#### Start container
Simply save the above **docker-compose.yml** file to your Containers directory and type in a terminal:
`docker-compose up`. This may take a bit of time as Docker downloads and installs the containers for the first time. Once completed on another tab you can check with: `docker container list`.

### Graylog Web Interface
Once Docker has finishes loading up, you can access the Graylog Web Interface. By default the server listens on port **9000** which is accessible both through `localhost` address and your system's LAN IP address.

Default Web Interface Username and Password: `admin:admin`

To add a new GELF input simply Navigate to: **System > Inputs**. As shown:

![alt text]({{page.image}}/graylog-new-input.png "Graylog New Input") 

And from the drop-down menu select `GELF TCP` and hit Launch new input. You'll then be asked to enter a title. More options are available like enabling TLS for transmitting logs securely, however since this is a quick guide it won't be required. 

#### Connection Test
Before continuing it's a good idea to test whether the newly created input is actually accepting our entries. To do this you could run **netcat** from a terminal:

```bash
echo -n -e '{ "version": "1.1", "host": "markuta.com", "short_message": "A short message", "level": 5, "_some_info": "foo" }'"\0" | nc -w1 127.0.0.1 12201
```

There now should be a new entry in `http://127.0.0.1:9000/sources`.

A more realistic approach would be doing the same test but from another computer on the same network, simply replace the IP address to your current system's (where Docker is installed) address. Be sure to allow or disable firewall rules on TCP port **12201**. As with Mac OS, the firewall will block incoming connections.

### Nzyme
Now that our centralised Graylog server is ready. It's time to set up Nzyme. Nzyme runs on Java and thus requires a Java Runtime environment. Both the OpenJDK or the Oracle JDK versions of Java 7 or 8 work well.

Check whether Java is installed:
`java -version`

To install OpenJDK Java 8:
`sudo apt-get install openjdk-8-jdk`

#### Download and Install
Download the most recent build from the [Releases](https://github.com/lennartkoopmann/nzyme/releases) page. At the time of writing the latest version was (v0.2). The use of deb package for Raspberry Pi or other Debian based distributions is strongly encouraged. However, the Jar file should work fine too.

Install package with dpkg tool: `sudo dpkg -i nzyme-0.2.deb`

Once installed the package creates multiple files and directories:

* `/etc/nzyme/` - configurations with an automatically generated sample
* `/var/log/nzyme/nzyme.log` - a log file for info/error information.
* `/usr/share/nzyme/` - contains the actual Jar file and symlinks

An example of **nzyme.conf** file:
```conf
# A name for this nzyme-instance.
nzyme_id = nzyme-sensor-1

# WiFi interface and 802.11 channels to use. Nzyme will cycle your network adapters through these channels.
# Consider local legal requirements and regulations. Default is US 2.4GHz band.
# Configure one or more interfaces here.
# See also: https://en.wikipedia.org/wiki/List_of_WLAN_channels
channels = wlan0:1,2,3,4,5,6,7,8,9,10,11,12,13,36,40,44,48

# There is no way for nzyme to configure your wifi interface directly. We are using direct operating system commands to
# configure the adapter. Examples for Linux and OSX are in the README.
channel_hop_command = sudo /sbin/iwconfig {interface} channel {channel}

# Channel hop interval in seconds. Leave at default if you don't know what this is.
channel_hop_interval = 1

# List of Graylog GELF TCP inputs. You can send to multiple, comma separated, Graylog servers if you want.
graylog_addresses = 192.168.1.113:12201

# There are a lot of beacon frames in the air. A sampling rate of, for example, 20, will ignore 19 beacons
# and only send every 20th to Graylog. Use this to reduce traffic. Set to 0 to disable sampling.
beacon_frame_sampling_rate = 0

```

All that needs to be changed is the interface device, if you're using something other than wlan0, and graylog_addresses which corresponds to the graylog server address. You can also change channels to suite your country's regulatory standards. The channels will hop on both 2.4GHz and 5GHz frequencies.

To start, stop, or check status about the service: 
* `sudo systemctl start nzyme`
* `sudo systemctl stop nzyme`
* `sudo systemctl status nzyme`

To enable the service to start automatically on boot: `sudo systemctl enable nzyme`

#### Sample Output
Example of Graylog running for :

![alt text]({{page.image}}/graylog-sample.png "Graylog Sample") 

### Notes
When navigating through the Graylog Web Interface, there may be some warning messages: _"Could not update field graph data"_ or _"Updating field graph data failed"_. I think those are down to the ElasticSearch configuration, which others have also experienced: [community.graylog.org](https://community.graylog.org/t/could-not-update-field-graph-data-errors/2012). You could change the update rate to every 30 minutes.

A big thanks to [Lennart Koopmann](https://github.com/lennartkoopmann) for creating and sharing this useful wireless tool.

