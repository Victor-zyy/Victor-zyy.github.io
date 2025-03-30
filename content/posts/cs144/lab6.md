+++
title = "cs144-lab6"
author = ["zyy"]
date = 2025-03-30T16:25:00+08:00
tags = ["c++", "cs144"]
categories = ["net"]
draft = false
showtoc = true
+++

## Router {#router}


### introduction {#introduction}

A router lies in IP layer of TCP/IP, when an ip-packet comes to the network interface, the router will route the ip-packet to another router (indirect) or another network(direct). But the router don't route the ip-packet randomly, it follow the routing tables strictly. The routing table is just a mapping for dst-ip and network interface. The router has the ability to drop the message if the ttl of the ip-header equals zero or one.


### routing table {#routing-table}

Let us talk about the routing table, and you might wonder how it looks like. I'll give a table for that from wiki to illustrate.

| network destination | Netmask     | Gateway     | Interface     | Metirc |
|---------------------|-------------|-------------|---------------|--------|
| 0.0.0.0             | 0.0.0.0     | 192.168.0.1 | 192.168.0.100 | 10     |
| 127.0.0.0           | 255.0.0.0   | 127.0.0.1   | 127.0.0.1     | 1      |
| 192.168.0.0         | 255.255.0.0 | 192.168.0.8 | 192.168.0.4   | 10     |

The **network destination** and **netmask** together describe the network identifier. The **gateway** contains the same information as the next hop, it points to the gateway through which the network can be reached. The **interface** indicates that what locally available interface is responsible for reaching the gateway. Actually in ip layer of the network, each interface has an ip address which is rational to understand. Sometimes the first line of the routing table is default entry with default gateway.


### longest-prefix match {#longest-prefix-match}

The route prefix and prefix length together specify a range of IP addresses (a network) that might include the datagram’s destination. The route prefix is a 32-bit numeric IP address. The prefix length is a number between 0 and 32 (inclusive); it tells the router how many most-significant bits of the route prefix are significant. For example, to express a route to the network “18.47.0.0/16” (this matches any 32-bit IP address where the first
two bytes are 18 and 47), the route prefix would be 305070080 (18 × 224 + 47 × 216 ), and the prefix length would be 16. Any datagram destined for “18.47.x.y” will match. If another one match with 18 prefix length, then the router must select the second one.


### data structure {#data-structure}

In this lab, we need a data structure to track the route table entry when added by os or tools(like iptables). I choose linked-list which is simple as my route table entry, here is what i did.

```c++

struct route_entry
{
  uint32_t route_prefix{};
  uint8_t prefix_length{};
  std::optional<Address> next_hop{};
  size_t interface_num {};
};
// inner data structure to store the route table
std::list<route_entry> route_table_ {};
```


## Outline {#outline}

![](/cs144/images/lab6_router.png)
The router just has several interfaces connected to it, the interface works in mac layer, so it will receive message from other router. That's why router will work to traverse every interface to find message that needs to be routed.
The router action is kind of easy to implement, you just traverse every interface to find messages available, then match the routing table, if matched, transmit message with the right interface.
