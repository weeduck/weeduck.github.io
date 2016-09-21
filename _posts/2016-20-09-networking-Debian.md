---
layout: post
title:  "Networking Debian"
date:   2016-09-20 18:10:28
categories: debian
---

# Networking Debian

This document explains how to configure and manage all aspects of the networking
stack for Debian.

## Contents
1. [Interface configuration](#interface-configuration)
  - [DHCP](#dhcp)
  - [Static IP](#static-ip)
  - [Interface Commands](#interface-commands)
  - [Subinterfaces](#subinterfaces)
  - [Bonding interfaces](#bonding-interfaces)
2. [Interface discovery](#interface-discovery)

## Interface configuration

One of the main thing we need to do is to configure the interfaces.
Here are some of the possible combinations in use.
All of the configurations for the network are in the /etc/network/interfaces file.

#### DHCP
If we want to have dhcp the config file should be:

~~~
# start the interface on reboot, or network service restart
auto eth0
# start the interface on plugging in the cable
allow-hotplug eth0
# DHCP
iface eth0 inet dhcp
~~~

#### Static IP

If we want static IP the file needs to be changed like this:
~~~
auto eth0
allow-hotplug eth0
# put the static options instead of dhcp
iface eth0 inet static
  address 172.16.0.7
  netmask 255.255.255.0
  network 172.16.0.0
  broadcast 172.16.0.255
  gateway 172.16.0.1
~~~

_*There can be ONLY one gateway interface in the whole config - otherwise the
network will not work.*_

#### Interface commands

If we need to route traffic through another interface, the best way would be
to add route instructions to the interface definition. The example of a setup
with two interfaces and route instructions is below:

~~~
auto eth0
allow-hotplug eth0
iface eth0 inet static
  address 172.16.0.7
  netmask 255.255.255.0
  network 172.16.0.0
  broadcast 172.16.0.255
  gateway 172.16.0.1

auto eth1
allow-hotplug eth1
iface eth1 inet static
  address 172.17.0.8
  netmask 255.255.255.0
  network 172.17.0.0
  broadcast 172.17.0.255
  #route instructions
	up route add -net 172.18.0.0 netmask 255.255.255.0 gw 172.17.0.1
	down route del -net 172.18.0.0 netmask 255.255.255.0 gw 172.17.0.1
~~~

#### Subinterfaces

If we need to have more IP addresses per interface the best way to do it is by
using subinterfaces:

~~~
auto eth0
allow-hotplug eth0
iface eth0 inet static
  address 172.16.0.7
  netmask 255.255.255.0
  network 172.16.0.0
  broadcast 172.16.0.255
  gateway 172.16.0.1
  up   ip addr add 172.16.0.8/24 dev eth0 label eth0:1
  down ip addr del 172.16.0.8/24 dev eth0 label eth0:1
~~~

#### Bonding interfaces

If we require port channel or ether channel it can be done by bonding interfaces.

We need to install the software for the bonding:
~~~
apt-get update
apt-get install ifenslave
~~~

And we need to create a bonded interface in the configuration:

~~~
auto bond0
iface bond0 inet static
  address 172.16.0.7
  netmask 255.255.255.0
  network 172.16.0.0
  broadcast 172.16.0.255
  gateway 172.16.0.1
	bond-slaves eth0 eth1
	bond-mode 802.3ad
	bond-miimon 100
	bond-downdelay 200
	bond-updelay 200
	bond-lacp-rate 1
	bond-xmit-hash-policy layer2+3
~~~

## Interface discovery

Debian discovers interfaces and remembers their name by their MAC address.  
In order to control this behavior we can edit a configuration file:

~~~
vim /etc/udev/rules.d/70-persistent-net.rules
########################################
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="c6:06:c5:2a:76:97", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="1a:9d:2a:35:f9:b9", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth1"

SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="92:9d:5e:bb:f2:c6", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth2"

SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="42:3a:94:d2:27:a6", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth3"

SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="ce:5b:cf:4e:72:12", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth4"
########################################
~~~
