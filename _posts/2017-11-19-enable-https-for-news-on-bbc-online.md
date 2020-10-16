---
layout: post
title: Enable HTTPS for News on BBC Online
date: 2017-11-19 09:45:47 +07:00
description: The BBC Online still forces users to use insecure HTTP by way of redirection. It's been over a year since BBC Online enabled HTTPS on their Homepage. However, deployment on certain paths are still is absent. The most relevant path being News, where the actual content resides.
tags: 
    - https
    - security
    - bbc
---
Since last April in 2016, the main BBC Homepage has been accessible only via HTTPS, which I thought was a good step forward, heading in the right direction. However, most pages or URLs still use insecure HTTP. Trying to navigate to a page while manually typing HTTPS in the browser address bar will force a 301 re-direct to HTTP.

Here's an example of cURL while navigating to `/news/` path:

```bash
$ curl -IL https://www.bbc.co.uk/news/technology
HTTP/1.1 301 Moved Permanently
Content-Type: text/html
Date: Sun, 19 Nov 2017 22:53:22 GMT
Location: http://www.bbc.co.uk/news/technology
Connection: Keep-Alive
Content-Length: 0

HTTP/1.1 200 OK
Server: Apache
Content-Type: text/html; charset=utf-8
X-News-Data-Centre: telhc
Content-Language: en-GB
X-PAL-Host: pal193.back.live.telhc.local:80
X-News-Cache-Id: 19441
Content-Length: 212175
Date: Sun, 19 Nov 2017 22:53:23 GMT
Connection: keep-alive
Set-Cookie: BBC-UID=c50curl/7.56.0; expires=Thu, 18-Nov-21 22:53:23 GMT; path=/; domain=.bbc.co.uk
Cache-Control: private, max-age=30, stale-while-revalidate
X-Cache-Action: MISS
X-Cache-Age: 0
X-LB-NoCache: true
Vary: X-CDN,X-BBC-Edge-Cache,Accept-Encoding
```

The `/news/` path and Homepage are probably the most visited in terms of network traffic compared to other aspects of the site, linking from various external sources (social media, news outlets, companies). It is strange to why they enabled HTTPS on their Homepage, and not every where else. It would of been good idea to create a beta version of news site for users to participate, which only allowed secure connections so that they can fine tune server configurations and learnt from data being collected.

More information is available from the BBC's Internet blog. An interesting post by Lead Technical Architect, Paul Tweedy, entitled: [Enabling Secure HTTP for BBC Online](http://www.bbc.co.uk/blogs/internet/entries/f6f50d1f-a879-4999-bc6d-6634a71e2e60) (July, 2016).  

It goes on to state the progress being made and also challenges faced for such large website.

* Technical & contractual changes to CDN (Contend Delivery Networks) partners.
* Impact of addition TLS encryption on CPU load and other computer resources
* Internal software changes (back-end development)
* Device support; Smart TV, iPlayer, Mobile, etc...

One user comments about the recent "Travel News" in the article:
> There's a slight irony of implementing HTTPS on the travel site when the proposed closure of that section of the BBC website was announced a few months ago. - Keith

It has been over **16 months** since that article was posted by Paul Tweedy. We're almost in 2018 and one of the most visited parts of BBC Online site still forces users to use insecure HTTP.

### Related news
Hacker News: [How the BBC News website has changed over the past 20 years ](https://news.ycombinator.com/item?id=15730218) (November, 2017)
