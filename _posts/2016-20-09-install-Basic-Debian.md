---
layout: post
title:  "Debian Basic Server Installation"
date:   2016-09-20 18:10:28
categories: debian
---

# Debian Basic Server Installation

This document describes the basic process for installing the Debian server.  
It covers only the basic options and creates a blank server which is expanded on
by other installations.

## Contents
1. [Install Debian](#install-debian)
  - [Manually](#manually)
  - [Using Preseed](#using-preseed)
  - [After Installation](#after-installation)
2. [Add users and configure SSH access](#add-users-and-configure-ssh-access)
  - [Add the users](#add-the-users)
  - [Install and secure the SSH system](#install-and-secure-the-ssh-system)
3. [Configuring the network](#configuring-the-network)

## Install Debian

The installation images can be found at the [debian project site](https://www.debian.org/distrib/netinst).  
The best image to use is the small amd64 image (net install).

There are two ways to install Debian on a server:

1. Manually
2. Using a preseed file

#### Manually

By using an installation image you can get a Debian installation up and running.  
It's best that you encrypt the disk only if it is warranted as it may decrease
performance.  
Do not use the whole disk, but resize if needed.

#### Using Preseed

By using the preseed file you can automate the installation of the Debian system.  

An example preseed file is attached:


~~~
### Localization
### Using the US keymap

d-i debian-installer/locale string en_US
d-i keyboard-configuration/xkb-keymap select us

### Network configuration

d-i netcfg/choose_interface select eth0
d-i netcfg/link_wait_timeout string 10

d-i netcfg/disable_autoconfig boolean true
d-i netcfg/disable_dhcp boolean true

d-i netcfg/dhcp_failed note
d-i netcfg/dhcp_options select Configure network manually

### You need to use dhcp if you are using a netinstall!!!
### The options below will be ignored

d-i netcfg/get_ipaddress string 192.168.89.90
d-i netcfg/get_netmask string 255.255.0.0
d-i netcfg/get_gateway string 192.168.1.1
d-i netcfg/get_nameservers string 8.8.8.8
d-i netcfg/confirm_static boolean true

d-i netcfg/get_hostname string testme
d-i netcfg/get_domain string viduck.com

d-i netcfg/hostname string testme.viduck.com

# Disable that annoying WEP key dialog.
d-i netcfg/wireless_wep string

d-i hw-detect/load_firmware boolean true

d-i mirror/country string manual
d-i mirror/http/hostname string http.us.debian.org
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string

# Suite to install.
#d-i mirror/suite string testing
# Suite to use for loading installer components (optional).
#d-i mirror/udeb/suite string testing

### Account setup

d-i passwd/root-login boolean false


# To create a normal user account.
### The username used can be anything, here it is ansible due to the possiblity
### of provisioning the system after install with an ansible playbook

d-i passwd/user-fullname string Ansible
d-i passwd/username string ansible
# Normal user's password, either in clear text
# mkpasswd -m sha-512 tghu76
d-i passwd/user-password-crypted password $6$1nqexRqjIhc$EK6Uc/X/35UjwQQmCs2uDn6GuwlQCcTGCqFtKrQTZHJeZ23nHzN93.pjjS/MLwV4teMNzyuc7c6RTS4pe91Zl1
d-i passwd/user-uid string 1010

# The user account will be added to some standard initial groups. To
# override that, use this.
d-i passwd/user-default-groups string

### Clock and time zone setup
# Controls whether or not the hardware clock is set to UTC.
d-i clock-setup/utc boolean true

# You may set this to any valid setting for $TZ; see the contents of
# /usr/share/zoneinfo/ for valid values.
d-i time/zone string Europe/Berlin

# Controls whether to use NTP to set the clock during the install
d-i clock-setup/ntp boolean true
# NTP server to use. The default is almost always fine here.
#d-i clock-setup/ntp-server string ntp.example.com

### Partitioning
## Partitioning example
# If the system has free space you can choose to only partition that space.
# This is only honoured if partman-auto/method (below) is not set.
#d-i partman-auto/init_automatically_partition select biggest_free

d-i partman-auto/disk string /dev/sda

d-i partman-auto/method string crypto
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

d-i partman-crypto/passphrase password tghu765rf
d-i partman-crypto/passphrase-again password tghu765rf


d-i partman-auto/choose_recipe select boot-crypto
d-i partman-auto-lvm/new_vg_name string crypt
d-i partman-auto/expert_recipe string boot-crypto :: \
	512 1024 512 ext4 $primary{} $bootable{} \
	method{ format } format{ } \
	use_filesystem{ } filesystem{ ext4 } \
	mountpoint{ /boot } \
	.\
	4096 8192 4096 linux-swap $lvmok{ } lv_name { swap } \
	in_vg { crypt } method{ swap } format{ } \
	.\
	10000 15000 20000 ext4 $lvmok{ } lv_name { root } \
	in_vg { crypt } method{ format } format{ } \
	use_filesystem{ } filesystem{ ext4 } mountpoint{ / } \
	.\
	10000 16000 20000 ext4 $lvmok{ } lv_name { storage } \
	in_vg { crypt } method{ format } format{ } \
	use_filesystem{ } filesystem{ ext4 } mountpoint{ /storage } \
	.\
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

## Controlling how partitions are mounted
# The default is to mount by UUID, but you can also choose "traditional" to
# use traditional device names, or "label" to try filesystem labels before
# falling back to UUIDs.
#d-i partman/mount_style select uuid

### Base system installation
# Configure APT to not install recommended packages by default. Use of this
# option can result in an incomplete system and should only be used by very
# experienced users.
#d-i base-installer/install-recommends boolean false

# The kernel image (meta) package to be installed; "none" can be used if no
# kernel is to be installed.
#d-i base-installer/kernel/image string linux-image-586

### Apt setup

### Package selection
tasksel tasksel/first multiselect standard, ssh-server

### GRUB installation
d-i grub-installer/only_debian boolean true

d-i grub-installer/with_other_os boolean true

# Due notably to potential USB sticks, the location of the MBR can not be
# determined safely in general, so this needs to be specified:
d-i grub-installer/bootdev  string /dev/sda
# To install to the first device (assuming it is not a USB stick):
#d-i grub-installer/bootdev  string default

# Alternatively, if you want to install to a location other than the mbr,
# uncomment and edit these lines:
#d-i grub-installer/only_debian boolean false
#d-i grub-installer/with_other_os boolean false
#d-i grub-installer/bootdev  string (hd0,1)
# To install grub to multiple disks:
#d-i grub-installer/bootdev  string (hd0,1) (hd1,1) (hd2,1)

# Avoid that last message about the install being complete.
d-i finish-install/reboot_in_progress note

# This will prevent the installer from ejecting the CD during the reboot,
# which is useful in some situations.
#d-i cdrom-detect/eject boolean false

#### Advanced options
### Running custom commands during the installation

### These commands change the network interface and add ssh keys for use by
### ansible during automated installation

d-i preseed/late_command string \
    in-target /bin/bash -c 'echo testme.viduck.com > /etc/hostname' ; \
		in-target /bin/bash -c 'printf "auto lo \niface lo inet loopback\n\nallow-hotplug eth0\niface eth0 inet static\n address 192.168.89.90\n netmask 255.255.0.0\n gateway 192.168.1.1\n dns-names 8.8.8.8\n " > /etc/network/interfaces' ; \
		in-target service networking restart ; \
		in-target apt-get update ; \
		in-target apt-get install htop ; \
    in-target /bin/bash -c 'mkdir -p /home/ansible/.ssh' ; \
    in-target /bin/bash -c 'echo "ssh-ed25519 WHATEVERISINTHEKEYSSHIDEALLYED25519 marko@FDSFDS" > /home/ansible/.ssh/authorized_keys' ; \
    in-target /bin/bash -c 'chown -R ansible:ansible /home/ansible' ; \
    in-target /bin/bash -c 'chmod 700 /home/ansible/.ssh' ; \
    in-target /bin/bash -c 'chmod 600 /home/ansible/.ssh/authorized_keys' ; \
		in-target /bin/bash -c 'echo "PasswordAuthentication no" >> /etc/ssh/sshd_config' ; \
    in-target /bin/bash -c 'sed -i "s/Port 22/Port 9090/" /etc/ssh/sshd_config'

~~~

After the preseed it is smart to change the user, root and hard drive passwords:

~~~
passwd USERNAME
passwd root
cryptsetup -y luksAddKey ENCRYPTED_PARTITION
cryptsetup luksRemoveKey ENCRYPTED_PARTITION
~~~

With this you are left with a fresh, blank system for further installation and
configuration.

### After installation

After the installation it is smart to install some basic packages:

~~~
apt-get purge exim4*
systemctl disable rpcbind
service rpcbind stop
apt-get update
apt-get install ntp htop vim screen tmux ufw \
mailutils postfix git tree sysstat zip unzip iotop
~~~

Change the hostname, hosts:

~~~
vim /etc/hostname
########################################
HOSTNAME
########################################

vim /etc/hosts
########################################
########################################

~~~

Change the postfix to only send using ipv4 (due to spam rules and SPF it
  is easier to only send mail over ipv4):
~~~
vim /etc/postfix/main.cf
#########################################
inet_protocols = ipv4
#########################################
~~~

Install apticron - software for notifying of updates on a server:

~~~
apt-get update

apt-get install apticron

vim /etc/apticron/apticron.conf
#########################################
EMAIL="admin@yourdomain.com"
#########################################
~~~

## Add users and configure SSH access

In order to access our new system we need to:

####Add the users:

~~~
adduser USERNAME
mkdir -p /home/USERNAME/.ssh

vim /home/USERNAME/.ssh/authorized_keys
#####################################
#THE SSH PUBKEY GOES HERE
#####################################

chown -R USERNAME:USERNAME /home/USERNAME/.ssh
chmod 700 /home/USERNAME/.ssh/
chmod 600 /home/USERNAME/.ssh/authorized_keys
~~~

The SSH key is generated by using either:

ED25519 - Elliptic curve
~~~
ssh-keygen -t ed25519
~~~

or RSA (at least 4096 bit, 8192 is fine)
~~~
ssh-keygen -t rsa -b 8192
~~~

The generated pubkey (\*.pub) is put into the authorized_keys file above

####Install and secure the SSH system:

~~~
apt-get update
apt-get install openssh-server
~~~

We need to edit the openssh config file:

~~~
vim /etc/ssh/sshd_config
#############################
Port 666
PermitRootLogin no
AllowUsers USERNAME1 USERNAME2
PermitEmptyPasswords no
PasswordAuthentication no
UsePAM no
##############################
~~~

And restart the ssh service:

~~~
systemctl restart ssh.service
~~~

You should add a small command to bashrc so that the admin is informed by mail
any time a user logs in to the system:

~~~
vim /etc/bash.bashrc
######################################################
# Send email report when someone logs in
echo "HOSTNAME - Shell Access on:" `date` `who` | mail -s "SSH-ACCESS on HOSTNAME" admin@yourdomain.com
######################################################
~~~

##Configuring the network

The explanation for how you should configure the network can be found at
[this link](networking-Debian.md).
