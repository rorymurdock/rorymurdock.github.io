---
layout: post
title:  "Virtual Mikrotik"
author: "Rory Murdock"
tags: Mikrotik Networking
---

Run a virtual Mikrotik with CHR

# Mikrotik CHR

[Mikrotik](https://mikrotik.com/) is a latvian router company that have created a great networking product that is solid, cheap, and reliable. I have used them for many years and they are absolutely solid.

They offer a product called [Cloud Hosted Router](https://wiki.mikrotik.com/wiki/Manual:CHR) (CHR) which is a virtualised copy of the RouterOS which supports up to 10GB routing and all of the normal features. I am lucky enough to have a CHR-P1 licence from my training but they're very [affordable](https://wiki.mikrotik.com/wiki/Manual:CHR#Paid_licenses). However there are plenty of other options out there

* [vyos](https://www.vyos.io/)
* [pfSense](https://www.pfsense.org/download/)
* [Sophos UTM](https://www.sophos.com/en-us/products/free-tools/sophos-utm-home-edition.aspx)
* [Untangle](https://www.untangle.com/get-untangle/)

First [download the CHR ova](https://mikrotik.com/download#chr)

The import the ova to ESX

![Screenshot]({{ site.url }}/assets/img/chr/ova_deploy.png)

![Screenshot]({{ site.url }}/assets/img/chr/ova_deploy_2.png)

Choose your main network

![Screenshot]({{ site.url }}/assets/img/chr/ova_deploy_3.png)

Finish and deploy the VM

![Screenshot]({{ site.url }}/assets/img/chr/ova_deploy_4.png)

Once it's deployed connect to the console. By default it has 1 CPU, 128MB of RAM and 64MB of disk. The specs are quite low but it's a router and that's more than enough for a lab, but you can adjust these if you need to.

![Screenshot]({{ site.url }}/assets/img/chr/chr_vm_status.png)

Login as `admin` and blank password

![Screenshot]({{ site.url }}/assets/img/chr/login.png)

![Screenshot]({{ site.url }}/assets/img/chr/splash.png)

You can get the DHCP address of the router using `/ip address print` or you can [download](https://mt.lv/winbox64) [winbox](https://wiki.mikrotik.com/wiki/Manual:Winbox) and use the neighbour function to find the IP of the router.

![Screenshot]({{ site.url }}/assets/img/chr/get_ip.png)

![Screenshot]({{ site.url }}/assets/img/chr/winbox.png)

You can use winbox or webfig for these steps.

Once logged in click Webfig.

![Screenshot]({{ site.url }}/assets/img/chr/webfig.png)

You'll see a list of ethernet interfaces

![Screenshot]({{ site.url }}/assets/img/chr/eth.png)

The first thing to do is change the password. Go to System -> Users

![Screenshot]({{ site.url }}/assets/img/chr/user.png)

Click password

![Screenshot]({{ site.url }}/assets/img/chr/user_2.png)

Enter your new password and click apply

![Screenshot]({{ site.url }}/assets/img/chr/user_3.png)

Next go to IP -> Services and disable everything except for winbox and www

![Screenshot]({{ site.url }}/assets/img/chr/disable_services.png)

From here you can set it up how you like, this is my basic lab config (double NAT) that has 8 VLANs with DHCP on each and traffic between them is isolated except VLAN 100 which is where my monitoring sits.

```shell
/interface ethernet
set [ find default-name=ether2 ] disable-running-check=no name=VLAN100
set [ find default-name=ether3 ] disable-running-check=no name=VLAN101
set [ find default-name=ether4 ] disable-running-check=no name=VLAN102
set [ find default-name=ether5 ] disable-running-check=no name=VLAN103
set [ find default-name=ether6 ] disable-running-check=no name=VLAN104
set [ find default-name=ether7 ] disable-running-check=no name=VLAN105
set [ find default-name=ether8 ] disable-running-check=no name=VLAN106
set [ find default-name=ether9 ] disable-running-check=no name=VLAN107
set [ find default-name=ether1 ] disable-running-check=no name=ether1-wan
/ip pool
add name=172.16.100.0/24 ranges=172.16.100.2-172.16.100.254
add name=172.16.101.0/24 ranges=172.16.101.2-172.16.101.254
add name=172.16.102.0/24 ranges=172.16.102.2-172.16.102.254
add name=172.16.103.0/24 ranges=172.16.103.2-172.16.103.254
add name=172.16.104.0/24 ranges=172.16.104.2-172.16.104.254
add name=172.16.105.0/24 ranges=172.16.105.2-172.16.105.254
add name=172.16.106.0/24 ranges=172.16.106.2-172.16.106.254
add name=172.16.107.0/24 ranges=172.16.107.2-172.16.107.254
/ip dhcp-server
add address-pool=172.16.100.0/24 disabled=no interface=VLAN100 name=172.16.100.0/24
add address-pool=172.16.101.0/24 disabled=no interface=VLAN101 name=172.16.101.0/24
add address-pool=172.16.102.0/24 disabled=no interface=VLAN102 name=172.16.102.0/24
add address-pool=172.16.103.0/24 disabled=no interface=VLAN103 name=172.16.103.0/24
add address-pool=172.16.104.0/24 disabled=no interface=VLAN104 name=172.16.104.0/24
add address-pool=172.16.105.0/24 disabled=no interface=VLAN105 name=172.16.105.0/24
add address-pool=172.16.106.0/24 disabled=no interface=VLAN106 name=172.16.106.0/24
add address-pool=172.16.107.0/24 disabled=no interface=VLAN107 name=172.16.107.0/24
/ip address
add address=172.16.100.1/24 interface=VLAN100 network=172.16.100.0
add address=172.16.101.1/24 interface=VLAN101 network=172.16.101.0
add address=172.16.102.1/24 interface=VLAN102 network=172.16.102.0
add address=172.16.103.1/24 interface=VLAN103 network=172.16.103.0
add address=172.16.104.1/24 interface=VLAN104 network=172.16.104.0
add address=172.16.105.1/24 interface=VLAN105 network=172.16.105.0
add address=172.16.106.1/24 interface=VLAN106 network=172.16.106.0
add address=172.16.107.1/24 interface=VLAN107 network=172.16.107.0
add address=172.16.0.1 comment=Fallback interface=ether1-wan network=172.16.0.1
/ip dhcp-client
add disabled=no interface=ether1-wan use-peer-dns=no
/ip dhcp-server network
add address=172.16.100.0/24 dns-server=172.16.100.1 gateway=172.16.100.1
add address=172.16.101.0/24 dns-server=172.16.101.1 gateway=172.16.101.1
add address=172.16.102.0/24 dns-server=172.16.102.1 gateway=172.16.102.1
add address=172.16.103.0/24 dns-server=172.16.103.1 gateway=172.16.103.1
add address=172.16.104.0/24 dns-server=172.16.104.1 gateway=172.16.104.1
add address=172.16.105.0/24 dns-server=172.16.105.1 gateway=172.16.105.1
add address=172.16.106.0/24 dns-server=172.16.106.1 gateway=172.16.106.1
add address=172.16.107.0/24 dns-server=172.16.107.1 gateway=172.16.107.1
/ip dns
set allow-remote-requests=yes servers=1.1.1.1
/ip firewall address-list
add address=10.0.0.0/8 list=private
add address=172.16.0.0/12 list=private
add address=192.168.0.0/16 list=private
/ip firewall filter
add action=accept chain=forward comment="Monitoring allow all" in-interface=VLAN100 out-interface=all-ethernet
add action=accept chain=forward comment="Monitoring allow ICMP" protocol=icmp src-address-list=private
add action=accept chain=forward comment="LAN in" src-address=192.168.0.0/24
add action=drop chain=forward comment="Drop inter VLAN traffic" dst-address-list=private log=yes log-prefix=drop_ src-address-list=private
/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether1-wan
/ip service
set telnet disabled=yes
set ftp disabled=yes
set ssh disabled=yes
set api disabled=yes
set api-ssl disabled=yes
/snmp
set enabled=yes
/system clock
set time-zone-name=Australia/Sydney
/tool graphing interface
add
/tool graphing resource
add
```