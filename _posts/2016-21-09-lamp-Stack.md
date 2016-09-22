---
layout: post
title:  "LAMP Stack Installation"
date:   2016-09-21 18:20:28
categories: debian
---

# LAMP Stack Installation

This document explains how to install a basic LAMP stack and how to secure it.

## Contents
1. [L is for Linux (or BSD)](#l-is-for-linux-(or-bsd))
2. [A is for Apache (or Nginx)](#a-is-for-apache-(or-nginx))


## L is for Linux (or BSD)

The choice of an OS or distro is not always simple. Sometimes it's a question
of capabilities but more often than not it is a question of preference.

The How To-s for some of the flavors:

1. Linux
  - [Debian]( {% link _posts/2016-20-09-install-Basic-Debian.md %} )
2. BSD


## A is for Apache (or Nginx)

The choice of a web server depends heavily on what exactly we want to serve.  
Whilst Apache is amazing at handling modules and being an all-in-one solution
Nginx is mush better at being a reverse proxy or "_front_" server for backend
servers.

The How To-s for some of the servers in contention:

1. [Apache2]({% link _posts/2016-21-09-servers-Apache2.md %})
2. [Nginx](servers-Nginx.md)
