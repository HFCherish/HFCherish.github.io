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

# 网关(gateway)

## what's gateway

Generally speaking, any entry to some 'network' is called gateway. So for programmers, there's api gateway, the entry to the backend-service network.

In the IP network context, gateway is used to connect two network.

> e.g. 
> 
> network A: 192.168.1.1 ~ 192.168.1.254,  mask: 255.255.255.0
> 
> network B: 192.168.2.1 ~ 192.168.2.254, mask: 255.255.255.0
> 
> A host a in **network A** wants to send data to a host b in **network B**:
> 
> 1. host a send data to gateway A(ip)
> 
> 2. gateway A send data to gateway B(ip)
> 
> 3. gateway B send data to host b(ip)

In the above case, 

1. there're two gateways. This is the connection-oriented gateway（面向连接的网关）. When the two networks are far and we create a gateway in each side, and then link them.
   
   - there  is also non-connection gateway（无连接的网关）

2- the gateway device must be able to route, which is in fact the router. Thus the ip of the gateway is often the ip of the router.
   
   - because it has to be about to route to the right gateway or host, thus the routing ability is required.
   
   - to route, the gateway in fact request the right ip of gateway/host from DNS, and then transfer the data there.

### explain with story

　　假设你的名字叫小不点，你住在一个大院子里，你的邻居有很多小伙伴，在门口传达室还有个看大门的李大爷，李大爷就是你的网关。当你想跟院子里的某个小伙伴玩，只要你在院子里大喊一声他的名字，他听到了就会回应你，并且跑出来跟你玩。

但是你不被允许走出大门，你想与外界发生的一切联系，都必须由门口的李大爷（网关）用电话帮助你联系。假如你想找你的同学小明聊天，小明家住在很远的另外一个院子里，他家的院子里也有一个看门的王大爷（小明的网关）。但是你不知道小明家的电话号码，不过你的班主任老师有一份你们班全体同学的名单和电话号码对照表，**你的老师就是你的DNS服务器**。于是你在家里拨通了门口李大爷的电话，有了下面的对话：

> 简单来讲，你老是就是你的域名服务器，你家小区的看门大爷就是你的网关。

小不点：李大爷，我想找班主任查一下小明的电话号码行吗？  
李大爷：好，你等着。（接着李大爷给你的班主任挂了一个电话，问清楚了小明的电话）问到了，他家的号码是211.99.99.99  
小不点：太好了！李大爷，我想找小明，你再帮我联系一下小明吧。  
李大爷：没问题。（接着李大爷向电话局发出了请求接通小明家电话的请求，最后一关当然是被转接到了小明家那个院子的王大爷那里，然后王大爷把电话给转到小明家）  
就这样你和小明取得了联系。

## default gateway

we can have multiple gateways. And when no gateway works, it goes the the default gateway.

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