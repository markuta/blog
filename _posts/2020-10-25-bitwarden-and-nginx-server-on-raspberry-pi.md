---
layout: post
title: Bitwarden and Nginx Server on Raspberry Pi
date: 2020-10-25
description: A how-to guide on setting up a self hosted Bitwarden server using a Raspberry Pi. This password management solution uses Docker and a Nginx reverse proxy. This guide also includes information on how to harden your Bitwarden set-up.
tags:
  - linux
  - raspberry pi
  - bitwarden
  - password
  - docker
  - nginx
  - configuration
  - secure
  - how-to
image: /assets/img/bitwarden-and-nginx-server-on-raspberry-pi/
---

# Overview

In this blog post I'll be covering how to install a self hosted Bitwarden server as a password management solution using Docker on a Raspberry Pi. We will get **two** containers running (Bitwarden server) and (Nginx reverse proxy). I'll also go into hardening the Bitwarden configuration and applying 2FA for log-ins.

![alt text]({{page.image}}bitwarden-login.png "Bitwarden login example")

## What is Bitwarden?

Bitwarden is an open-source password management solution. It supports almost all major systems. The version we're going to be using is the unofficial one created by Daniel Garcia, Github page: https://github.com/dani-garcia/bitwarden_rs. This version of Bitwarden is unofficial but it's really well made, and just works.


# Requirements

* Raspberry Pi (I'm using a model 3 B+)
* Docker software
* Bitwarden_rs (unofficial version)
* Domain name for TLS certificate

### Optional

* [Zymkey 4i](https://amzn.to/37HlgVB) is a Hardware Security Module for RPi.


# Installation

To start off with you'll want to download and install the latest version of Raspbian on your Pi. I personally recommend **Raspbian Buster Lite** (now called Raspberry Pi OS Lite), since it will be running 24/7 as a server, you don't really need a desktop environment nor the default office suite packages that are included. Make sure that the device is connected to the internet and contains the latest packages, I also like to enable SSH during the initial installation process and harden the `sshd_config` configuration file.

I will cover how to install **Zymbit zymkey 4i** IoT security module in a future post.

## Docker

We are going to be running BitWarden as a Docker container. Docker makes it an easy and simple to manage containers, which we can easily upgrade in the future. The image we are going to be use is available on https://hub.docker.com/r/bitwardenrs/server. 

Download and install Docker software with following on the Pi:

```bash
sudo curl -fsSL get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

Give the user permission to run Docker (`pi` is the default user):

```bash
sudo usermod -aG docker pi
```

Make sure Docker start on every system boot:

```bash
sudo systemctl enable docker
```

Restart your Raspberry Pi

```bash
sudo reboot
```

Once restarted, your Raspberry Pi should be ready to move onto with the configuration. 



# Configuration

Now that we have all the necessary applications installed we can continue with the configuration. We will  first set up a Bitwarden container, as well as the Nginx reverse proxy container. Later on we'll configure a Dockerfile to start all containers at once, I will be using a custom `docker-compose` file, found [here](#dockerfile).

A quick overview of what we're going to do:

* Pull the latest `bitwarden_rs` image from Docker hub
* First Start-up
  - create a new account
  - enable two-factor authentication
* Stop the container
  - disable new registrations
  - disable admin panel
  - enable HTTPS support
* Start the container with the new options + nginx


### Pulling image from Docker Hub

The Docker image we're going to use is by [https://hub.docker.com/r/bitwardenrs/server](https://hub.docker.com/r/bitwardenrs/server). You can find the source code on [https://github.com/dani-garcia/bitwarden_rs](https://hub.docker.com/r/bitwardenrs/server). You also no longer need to use the tag `bitwardenrs/server:raspberry` for Raspberry Pi systems.

To pull the image with Docker:

```bash
docker pull bitwarden_rs/server:latest
```



## First Time Start-up

After downloading the docker image you would want to choose a folder to mount a volume on the host system for persistent storage. The directory that I have chosen is located `/bw-data`. This is where all of our encrypted passwords will be stored, along with other web files. 

To run the container for the first time:

```bash
docker run -d --restart always \
 --name bitwarden \
 -e SIGNUPS_ALLOWED=true \
 -v /bw-data/:/data/ \
 -p 60888:80 \
 bitwardenrs/server:latest
```

Your Bitwarden web server will be accessible at: `http://IP-ADDRESS>:60888`. You can change the external port number by modifying the previous command (`-p`). Go ahead and register an account and log-in. To enable 2FA follow the steps below.

Go to **Settings**:
![alt text]({{page.image}}bitwarden-2fa-settings.png "Bitwarden 2FA Settings")

Select **Two-step login** and the type of 2FA you want to use. For example Authenticator app:
![alt text]({{page.image}}bitwarden-2fa-settings-2.png "Bitwarden 2FA Settings 2")

Then enter your code. You can now stop the container and move on to the next stage. Locking down your Bitwarden server and including a Nginx web server and Fail2ban.
```
docker stop bitwarden
```
## Hardening Process

In the next step we'll be going through the process of hardening our server for actual use. We'll be covering how to set up a Nginx reverse proxy and also install a certificate. 

To keep things organised I've created a folder called **bitwarden** which stores all configuration files and folders, the structure looks like this:
```
- bw-data/
- nginx/
  - nginx.conf
  - ssl.conf
  - dhparams.pem
- docker-compose.yml
```

##### Dockerfile

This Dockerfile was created to ease the installation process. It contains three containers with some configuration options. You will have to change these to suite your own environment. The environment variables for the Bitwarden container are for my own personal preference.

```yaml
version: "3.5"
services:

  nginx:
    restart: always
    image: nginx:stable-alpine
    container_name: nginx
    volumes:
      - ./nginx/dhparams.pem:/etc/ssl/dhparams.pem
      - ./nginx/ssl.conf:/etc/nginx/ssl.conf
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/cache/:/etc/nginx/cache
      - ./nginx/error.log:/etc/nginx/error.log
      - /etc/letsencrypt:/etc/letsencrypt
      - /etc/ssl/certs/self-signed.crt:/etc/ssl/certs/self-signed.crt
      - /etc/ssl/private/self-signed.key:/etc/ssl/private/self-signed.key
    ports:
      - "60888:60888"
    networks:
      - bit_net

  bitwarden:
    restart: always
    image: bitwardenrs/server:latest
    container_name: bitwarden
    volumes:
      - ./bw-data:/data
    environment:
      - TZ=Europe/London
      - LOG_FILE=/data/bitwarden.log
      - EXTENDED_LOGGING=true
      - LOG_LEVEL=warn
      - ROCKET_WORKERS=20
      - WEBSOCKET_ENABLED=true
      - SIGNUPS_ALLOWED=false
      - DISABLE_ADMIN_TOKEN=true
      - INVITATIONS_ALLOWED=false
      - SHOW_PASSWORD_HINT=false
      - DISABLE_ICON_DOWNLOAD=false
    ports:
      - "80"
      - "3012"
    networks:
      - bit_net

networks:
  bit_net:
    name: bit_net
```


##### nginx.conf

The **nginx.conf** file I use for the reverse proxy for Bitwarden. Within each server configuration update `listen 60888` and `server_name bitwarden.example.com;` to suit your own preference. You can leave the rest as it is.

```conf
http {
  error_log /etc/nginx/error.log warn;
  client_max_body_size 20m;
  server_tokens off;

  # Use self-signed certificate for IP addresses
  server {
      listen 60888 default_server ssl http2;
      server_name _;
      server_name_in_redirect off;
      return 404; # deny all requests
      ssl_certificate /etc/ssl/certs/self-signed.crt;
      ssl_certificate_key /etc/ssl/private/self-signed.key;

  }

  # The main Bitwarden web config 
  server {
      listen 60888 ssl http2;
      server_name bitwarden.example.com;
      include /etc/nginx/ssl.conf; # valid certificate
      client_max_body_size 128M;

      location / {
          proxy_pass http://bitwarden:80;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }

      location /notifications/hub {
          proxy_pass http://bitwarden:3012;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
     }

      location /notifications/hub/negotiate {
          proxy_pass http://bitwarden:80;
      }
  }

}
```

##### ssl.conf

This file will be included by the previous **nginx.conf**. You need to replace the options `ssl_certificate`, `ssl_certificate_key`, and  `ssl_trusted_certificate` to suit your own domain name. 

```conf
ssl_certificate      /etc/letsencrypt/live/example.com/fullchain.pem;
ssl_certificate_key  /etc/letsencrypt/live/example.com/privkey.pem;

# Improve HTTPS performance with session resumption
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;

# Enable server-side protection against BEAST attacks
ssl_protocols TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384";

# RFC-7919 recommended: https://wiki.mozilla.org/Security/Server_Side_TLS#ffdhe4096
ssl_dhparam /etc/ssl/dhparams.pem;
ssl_ecdh_curve secp521r1:secp384r1;

# Additional Security Headers
# ref: https://developer.mozilla.org/en-US/docs/Security/HTTP_Strict_Transport_Security
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

# ref: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
add_header X-Frame-Options DENY always;

# ref: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
add_header X-Content-Type-Options nosniff always;

# ref: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection
add_header X-XSS-Protection "1; mode=block" always;

# Enable OCSP stapling
# ref. http://blog.mozilla.org/security/2013/07/29/ocsp-stapling-in-firefox
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
resolver 1.1.1.1 1.0.0.1 [2606:4700:4700::1111] [2606:4700:4700::1001] valid=300s; # Cloudflare
resolver_timeout 5s;
```

##### dhparams.pem
To generate a **4096-bit** Diffie-Hellman parameter with openssl, type: 
```bash
openssl dhparam -out dhparams.pem 4096
```

#### Certificates

> DO NOT USE THE DEFAULT HTTP PORT FOR YOUR PASSWORD MANAGEMENT! 

To use the official Bitwarden app on say an iPhone with your self-hosted environment you need to use a valid TLS certificate. If you don't the OS will throw an error and refuse the connection since the certificate isn't valid. A workaround may be to add your self-signed certificate (not tested) to the trusted list on each device. A better approach would be to generate a valid TLS certificate.

For Let's Encrypt there are two main methods of verification (excluding TLS-ALPN-01): `HTTP-01` and `DNS-01`. If you're like me with an ISP that uses a heavily NATed  network then you can't really use the first option. So I'll be using second option which requires a domain name.

Download and install **certbot** with:

```bash
sudo apt-get install certbot
```

Run **certbot** with DNS as the preferred challenge:

```
certbot --manual --preferred-challenges dns certonly -d '*.example.com'
```

I'd recommend you to obtain a **wildcard certificate** instead of a single subdomain certificate. This way you don't need to reveal your Bitwarden server to the world, since there's a public record of every Let's Encrypt registered certificate.


## Thanks
[Bitwarden](https://bitwarden.com/) for creating an awesome password management solution.

[Dani Carcia](https://github.com/dani-garcia/bitwarden_rs/) for creating a port of Bitwarden.

[Let's Encrypt](https://letsencrypt.org/) for free certificates for everyone.
