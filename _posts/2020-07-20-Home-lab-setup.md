---
layout: post
title:  "Home Lab Setup"
author: "Rory Murdock"
tags: Lab
---

How I've setup my Home Lab

# Home labs

After not having a proper lab for years I decided to splurge out and get one so I can do some learning and reduce my collection of old mac minis and headless laptops.

There were a few requirements

* Had to be on premise
* Small and portable
* Low power consumption
* Run ESXi

 I wanted something local that I could leave running without incuring costs, but I also wanted to be able to take it around with me as I travel a bit. a 2RU used server would use far too much power and be a pain to get shipped here or take around with me.

 A micro PC was the best way to go there's a few good options out there such as

 * Intel NUC 9 Extreme
 * Intel NUC 10 i7
 * HP Z2 Mini G4
 * Mac Mini

In the end I chose the NUC10 i7 with 64GB or RAM and a 2TB SSD as it was the best performance for the price, small, and very low power consumption.

![Screenshot]({{ site.url }}/assets/img/homelab/boxes.png)

Setup was pretty straight forward, I installed my RAM, m2 SSD, the Ultra fit USB. Put the ESX image on a USB for installation, I did have to [modify my ESX image to add in a NIC driver](https://www.virten.net/2020/03/esxi-on-10th-gen-intel-nuc-comet-lake-frost-canyon/).

You could follow this [full guide](https://andrewroderos.com/vmware-esxi-home-lab-intel-nuc-frost-canyon/) if you need instructions on how to install ESXi.

![Screenshot]({{ site.url }}/assets/img/homelab/running.png)

Little tip: If you have an Apple Magic keyboard you can use it as a USB keyboard if you plug it in.

The basic outline of what to do:

* Setup vSwitch and port groups for VLANs
* Setup CHR
* Add NATing
* Configure DHCP
* Configure VLANs
* Security
* Deploy VMs

## Setup vSwitch and port groups for VLANs

First we need to add a vSwitch for the VLANs, now if we had a physical router we would have a second NIC to act as the VLAN Trunk into our vSwitch however as will be running a virtual router we'll create the isolated vSwitch with the VLAN port groups each attached to an interface on the CHR.

Create the vSwitch

![Screenshot]({{ site.url }}/assets/img/homelab/vswitch.png)

Then create a port group for each VLAN you want

![Screenshot]({{ site.url }}/assets/img/homelab/port_group.png)

![Screenshot]({{ site.url }}/assets/img/homelab/port_group_overview.png)

Then add an interface per VLAN to the CHR

![Screenshot]({{ site.url }}/assets/img/homelab/nics.png)

This is the topology of the VLAN vSwitch

![Screenshot]({{ site.url }}/assets/img/homelab/topology.png)

## Setup CHR

Complete the [CHR installation]({% post_url 2020-07-21-Mikrotik-CHR %}) first then

## Deploy VMs

Some of the things I'm running on my homelab are

* [PRTG](https://www.paessler.com/prtg)
* [Zabbix](https://www.zabbix.com/)
* [Docker](https://ubuntu.com/download)
* [Kubernetes](https://kubernetes.io/)
* [ELK](https://www.elastic.co/what-is/elk-stack)
* [Windows Codespace](https://docs.microsoft.com/en-us/visualstudio/codespaces/how-to/self-hosting-vscode)
* [Ubuntu Codespace](https://docs.microsoft.com/en-us/visualstudio/codespaces/how-to/self-hosting-vscode)
* [Veeam](https://www.veeam.com/blog/backup-replication-community-edition-features-description.html)
