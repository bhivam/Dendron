---
id: zm6fntegp8chmzx92sfd094
title: Theory of Networks
desc: ''
updated: 1680569585077
created: 1680566154761
---

#### Link
a small, local network where every device on a the link can directly communicate with every other device. 


#### Hub
A good example of this would be with Ethernet. Devices are connected to a hub. The hub will forward all messages sent to it, to all devices. Then devices will ignore messages not intended for them.

If multiple devices send messages at the same time, they interfere with -> both devices wait a random amount of time and resend. 

Messages are packet-based where each packet have a maximum size and each packet has the MAC address of the sender and recipient.

###### Note: 
Wifi is a wireless version of ethernet which itself is a wired version of a wirless medium called aloha net.

##### Problem with Hub Design
Hub design gets unwieldy as the number of devices/network traffic grows. n^2 problems happens here since every port needs to be connected to every other. Now to solve this we come up with e switch.

#### Switch
A switch allows us to divide the link into subsets. The switch is connected to multiple devices (think hubs). it tracks what addressses are in use for each port. Messages are only sent to the port that handles that address. Now we don't have as much of n^2 problem since the switch will send it only to the correct (how?) hub.

A switched or unswitched network is usually run by a single organization. This has very little intrisic security since every device can here every other device. Anyone can watch the packets going in and out of any other device on the network. Because of this modern wifi routers will route information to each device independently rather than blasting packets to every device on the network.

#### Interwork Communication
Internetworks allow us to communicate with devices outside of our network. 

The biggest internetwork is the Internet!

An internetork is a network of network and the way they function is with the use of routers. Routers are the layer between your network and outside networks. When they recieve a message meant for a device outside the network, they will pass it on. 

##### How does this work?

We identify devices by IP addresses. IP assumes every device has a unique address. The problem is that there are not enough addresses. 

###### How do we get around this?
Typically home users only get one IP address. So how can multiple devices connect to other devices on the internet? Modern routers us Network Address Translation. 

##### NATs
A NAT is a peice of hardware which is either standalone or in a router.
- Observes connections
- forward connections to appropriate devices
- allows entire network to pretend to be device because connections are forwarded to the correct devices and from a given device to the correct outside machine. 

But this means its really diffiuclt to run servers? How does the NAT know where to forward server connections to?
- Some NATs let you specify devices to handle connection requests for specific ports, so you can run servers from those devices. 

##### How do messsages get from my router to their destination
I will send my packet to the router. Then the router will decide whether to send it back to the network or if it needs to send it to a different router which gets it closer to the intended destination.

Routers have a protocl which use routing tables or simply routes to determine where to send messages intended for external networks. 

IP addresses are clustered in groups. So addressed controlled by a single entity are considered a "subnet". A subnet is a group of IP addresses that share a prefix. Originally, IP routing used 8- 16- or 24-bit prefixes. So this are class A B and C prefixes respectively, A gets 2^24 bit suffixes, B gets 2^16 bit suffexes and C gets 2^8 bit suffixes. 

A is absurdly large and C is too too small, so now we have classess routing where prefixes are of any length. Subnets are indicated by giving an IP address along with prefix length in bits e.g. 1.2.3.4/8. 

Routing tables say where to send messages for all known subnets. Routers have protocols within the internet to communicate routing information to each other. 

If part of the network goes down, routers will "route around" the damage and find new routes.