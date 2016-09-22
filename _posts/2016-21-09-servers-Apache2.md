---
layout: post
title:  "Apache2 server"
date:   2016-09-21 18:20:28
categories: debian
---

# Apache2 server

This document describes some of the intricacies of the Apache2 server, and
its possible uses.

## Contents
1. [Installation](#installation)
  - [Debian installation](#debian-installation)
2. [Vhosts](#vhosts)

## Installation

#### Debian installation:

Installing apache2 on debian is simple:

~~~
apt-get update
apt-get install apache2 apache2-doc
~~~

If we need some modules enabled it is done like this:

~~~
a2enmod rewrite ssl
service apache2 restart
~~~

Securing apache:

~~~
vim /etc/apache2/conf-available/security.conf
#########################################
ServerTokens Prod
ServerSignature Off
#########################################
~~~

## Vhosts

Setting Vhosts (virtual hosts) is a much needed option in apache2.

In this section we will explain how you can set your own Vhost with some more
specific options. This section is written for debian type OS, but you can adapt
it to your distro. The configs are the same, only the locations may differ.

Let us begin by creating the directories for our websites:

~~~
mkdir -p /var/www/test.com/html/
mkdir -p /var/www/example.com/html/

chown -R $USER:$APACHE_USER /var/www/
chmod -R 755 /var/www/
~~~

Lets create index.html files for our websites as:

~~~
vim /var/www/example.com/html/index.html
############################################
<html>
  <head>
    <title>Welcome to the apache example.com!</title>
  </head>
  <body>
    <h1>Success!  The example.com virtual host is working!</h1>
  </body>
</html>
############################################


vim /var/www/test.com/html/index.html
############################################
<html>
  <head>
    <title>Welcome to the apache test.com!</title>
  </head>
  <body>
    <h1>Success!  The test.com virtual host is working!</h1>
  </body>
</html>
############################################
~~~

Now we need to create the Vhost files in the /etc/apache2/sites-available/ dir:

~~~
vim /etc/apache2/sites-available/example.com.conf
#################################################

<VirtualHost *:80>

    Server example.com
    ServerAlias www.example.com

    ServerAdmin admin@example.com
    DocumentRoot /var/www/example.com/html

    ErrorLog ${APACHE_LOG_DIR}/example.com.error.log
    CustomLog ${APACHE_LOG_DIR}/example.com.access.log combined

</VirtualHost>

#################################################
~~~

~~~
vim /etc/apache2/sites-available/test.com.conf
#################################################

<VirtualHost *:80>

    Server test.com
    ServerAlias www.test.com

    ServerAdmin admin@test.com
    DocumentRoot /var/www/test.com/html

    ErrorLog ${APACHE_LOG_DIR}/test.com.error.log
    CustomLog ${APACHE_LOG_DIR}/test.com.access.log combined

</VirtualHost>

#################################################
~~~

And enable the Vhosts:

~~~
a2ensite test.com.conf example.com.conf
apachectl -t
service apache2 restart
~~~

