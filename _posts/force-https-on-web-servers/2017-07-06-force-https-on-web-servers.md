---
layout: post
title: How to Force HTTPS on Web Servers
date: 2017-07-06 01:00 +0700
description: How to configure Web Servers (Nginx, Apache, IIS, OpenLitespeed and Lighttpd) to Force HTTPS by default.
tags: 
    - configuration
    - secure
    - https

---
I've seen plenty of websites that use https but don't force it by default, this isn't considered a good security practice and should be resolved promptly. Below lists five of the most popular web servers (Nginx, Apache, IIS, OpenLitespeed and Lighttpd) configurations to force HTTPS by default.

All tests were carried out on a local Debian Stretch server with the exception of IIS.

> All **http://** requests will be (301) Moved Permanently to **https://** with respected request path.

### Nginx
Tested version 1.6.2. Configuration file **/etc/nginx/sites-enabled/example.conf** within the `server{ }` section:

```
server {
    listen      80;
    server_name www.example.com;
    return 301 https://$server_name$request_uri;
}
```

### Apache
Tested version 2.4.10. Configuration file **/etc/apache/sites-enabled/httpd.conf** within the `<VirtualHost>` section:

```
<VirtualHost *:80>
   ServerName www.example.com
   DocumentRoot /var/www/html
   Redirect / https://www.example.com/
</VirtualHost>
```

### IIS
May require an additional module to be installed, more details available over at [Microsoft's guide](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module). Configuration file **web.config** within the `<rewrite>` section:

```
<rewrite>
	<rules>
		<rule name="Force https" enabled="true" patternSyntax="Wildcard" stopProcessing="true">
			<match url="*" negate="false" />
			<conditions logicalGrouping="MatchAny">
				<add input="{HTTPS}" pattern="off" />
			</conditions>
			<action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" redirectType="Found" />
		</rule>
	</rules>
</rewrite>
```

### OpenLiteSpeed
Tested version 1.4.26. Configuration file **/usr/local/lsws/conf/vhosts/Example/vhconf.conf** within the `rewrite { }` section. Alternatively edit settings through the provided WebAdmin (on port 7080) by navigating to **Virtual Hosts** > **Rewrite** > **Rewrite Rules** and add the following:

```
RewriteCond %{HTTPS} !on
RewriteRule ^(.*)$ https://%{SERVER_NAME}%{REQUEST_URI} [R,L]
```

### Lighttpd
Tested version 1.4.35. Configuration file **/etc/lighttpd/lighttpd.conf**. The following will apply to all vhosts:

```
$HTTP["scheme"] == "http" {
    # Apply to all vhosts
    $HTTP["host"] =~ ".*" {
        url.redirect = (".*" => "https://%0$0")
    }
}
```
