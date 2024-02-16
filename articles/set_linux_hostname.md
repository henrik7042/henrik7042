Setting host and domain name in a installed Linux system
========================================================

This article will go through the steps for setting or changing
the host name and domain name.

The description apply for Debian 12 (Bookworm) but can probably
be valid for other systems as well.

Test
----
There are several ways to get the host and domain name and
this the ways that will be cover:
```
$ hostname
fido
```
```
$ hostname --fqdn
fido.example.com
```
```
$ domainname
example.com
```
```
$ hostnamectl hostname
fido
```
Changes
-------

Three files are involved `/etc/hostname`, `/etc/sysctl.d/local.conf` and `/etc/hosts`.

The file `/etc/sysctl.d/local.conf` does propably not exist so it need to be created.

```
$ cat /etc/hostname
fido
```
```
$ cat /etc/sysctl.d/local.conf 
#
# /etc/sysctl.conf - Configuration file for setting system variables
# See /etc/sysctl.d/ for additional system variables.
# See sysctl.conf (5) for information.
#

kernel.domainname = example.com
```
```
$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       fido.example.com fido

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
Replace fido and example.com parts in the above files and reboot.
