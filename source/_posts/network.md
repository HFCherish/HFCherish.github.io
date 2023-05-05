---
title: network
toc: true
tags:
  - network
date: 2020-07-23 14:10:25
---


[Eli the computer guy](https://www.youtube.com/watch?v=rL8RSFQG8do&list=PLF360ED1082F6F2A5)

# Introduction

the whole picture 

Speed & storage unit

Physical & logical

modem

* t1
* dsl:
  * no faster than 12Mb/s
  * Asynchronous: download faster than upload
* cabel
* satellite

Router

firewall

* block the internet to get into your network

VPN

* enable the internet to get into your network
* client-server

Switch

* Connect everything together

# TCP/IP

ip: internet protocol. How to find a computer.

tcp: transmission control protocol. How to communicate between 2 computers

suite: tcp protocol, ip protocol, etc

tcp windowing

components:

* ip address

* subnet

* default gateway

* dns

* dhcp

* nat

subnet mask: network identifier, device identifier.

```shell
# for windows, first release and renew to ensure to get the latest configurations
ipconfig /release
ipconfig /renew
ipconfig /all

# for linux and mac
ifconfig [-a]

# ping 10.1.10.11 10 times with 60ms ttl
ping www.baidu.com -c 10 -m 60

# in windows, it's tracert. For linux and mac, it's traceroute
traceroute www.baidu.com
```

## OpenDNS for network security

dns can be hacked so that you're redirected to some unwanted websites. Or a virus on your computer tries to pull more viruses  from some virus website. 

OpenDNS is a product to prevent such nasty issues.

1. With a router, and dhcp on that router (allocating ip/subset mask,etc to all your network devices), you config the dns as the opendns

2. On opendns control panel, add your network (the external ip of your network, i.e. the router ip address) and config the filter for your network.

3. When accessing unallowed websites, you will be redirected to a block opendns page by opendns.

If you don't have static ip, install the dynamic ip address from opendns, which will tell the opendns your current network address.

# Servers

server operating system: more robust and expensive, compared to desktop operating system.

specific hardware: for data center. xeon processor, redundant power supply, raid (redundant hard drive).

# Voice over ip (VOIP)

At the beginning, telephone system communicate using wires. It's completely separate to comupter system.

VOIP make the audio transimit through the TCP/IP protocol.



Client-Server infrastructure: hard phones / soft phones have to install the VOIP client to communicate to VOIP server, which routes all audio communications.

Iphone in fact installs VOIP client and phone through VOIP service. IPhone is the above soft phones.



Gateways. It can connect different communication system, e.g. the old telephone system and VOIP system, so that you can call through VOIP server -> Gateway -> the wireline telephone.



Codec. It encode the audio transmitted on VOIP service and decide what the packet should be like. So it decides the quality and bandwidth it needs.



QOS. quality of service. On switches and routers, config the VOIP transmision as high priority, so that it won't be influenced by other bandwidth sharer.



Unifed communication. I think it's the whole idea that make the telephone and computer stay in the same system, so that we can do a lot by this.

# Cloud Computing

Web application. If we want some application, but it's in fact not on our own hardware, and it's outside somewhere, then it's in fact cloud computing. It's just maybe it's private cloud.



Cluster. And to be a cloud, there also needs to be a cluster, so that when some bad things happen, it can recover itself.

Cluster make the services more robust. It's a normal way for cloud to provide robustness.



Terminal Services. There's terminal service server and thin clients. You can access a remote operating system desktop through thin clients. This seems to be the remote connection. Furthermore, the remote system can just be an specific application instead of a desktop. In such case, there's application server.

This is an old technique. In old days, it's called mainframe and dumb terminals. Anyway, the thin clients capture all the strokes you made and send them to the terminal service server, and the terminal service server sends the result image back. The teminal service server is the one with processing abilities, and all thin clients share the cpu time, each allocated with one slice of CPU time.

Terminal service is also a cloud related technique to access cloud services.



Virtualization. A tenichque that separate the operating system from the hardware, so that you can transfer the operating system with the applications in it easily. There're 2 flavours to implement it:

1. client installer. This is the normal virtualbox or vmware fusion. You installed this installer on your current operating system, and then you installed another os box using the installer.

2. hypervision. Hypervision is in fact an operating system. Install hypervision on hardware, and then using the management client on your computer to install anything you want on that hypervision. The hypervision will then allocate a piece of hard drive, ram, etc to the os you asked. In the vmware world, the hypervision is esxi which is always free, the management software is the vsphere which is in charged.

Current cloud computing should be using some techniques like the hypervision. Anyway, they try to separate the os from the hardware, so that you can copy-paste the whole environment easily to another hardware like a file.