---
title: VPN
toc: true
tags:
  - network
date: 2020-05-06 19:14:51
---

[the great video](https://www.youtube.com/watch?v=q4P4BjjXghQ)

# History

## Why does we create internet?

It's created from the need of American military, to protect the communication in the wars. 

The old communication system, phone, connects Lily with Tom through fixed central offices. If some of the central offices are destroyed by nuclear, then the rerouting of the communication line is difficult, that the commnunication will fail.

Ta Da.. Internet comes. It communicates through millions of routers. Even half of the routers are destroyed, there may still be the way to communicate.

## Why does we create VPN (virtual private network)?

Internet way secure the communication from high level, however, the data from a computer are in danger now. If a hacker can get all the data from a router, then data of users are leaked since it will go through the router.

To protect the communication from one to another, you can build physical private line. However, it's too expensive, and [VPN](https://en.wikipedia.org/wiki/Virtual_private_network) provides another way. The VPN secure the data in the following ways:

1. creates tunnel between two communicators, 
2. the data in the tunnel is encrepted
3. if the tunnel detects attack, it will destroy the old tunnel & create a new one.

Client-Server infrastructure. We need to know:

1. external address (the address of vpn server)

2. username & password

These factors may influence the VPN quality:

1. hacker penetrate. (like said above, during the rebuilt of tunnel, packets maybe lost)

2. old wiring, which will cause frequent tunnel rebuilt, too

3. old routers that not allow vpn passing through
