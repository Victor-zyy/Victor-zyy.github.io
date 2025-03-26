+++
title = "cs144-lab5"
author = ["zyy"]
date = 2025-03-26T19:59:00+08:00
tags = ["cs144", "c++"]
categories = ["net"]
draft = false
showtoc = true
+++

## Object {#object}

In this lab5, I am gonna to implent an IP interface for TCP/IP stack and IP router later in lab6. The IP interface is used to send etherframe using payload from IP layer. So, in this layer, we handle the mac etherframe in details. If we don't konw the mac address of the next-hop IP address, we need to  send ARP request to this subnet, then queuing the datagram until the ARP message received. After receiving the ARP message, we need to doc the &lt;next-hop, mac&gt; pair mapping for 30 secs in this lab, with this mapping, the next time sending to the same ip would not send ARP message again.

The most important thing we need to know that, the ARP protocol is based on the etherframe (aka MAC layer), which in another words to say, the ARP message is the payload of the etherframe except that it has special meaning of each field of the bytes.

Ok, Let's see the ARP (address resolution protocol) in wiki. it says that ARP enables a host to send an IPv4 packet to another node in the local network by providing a protocol to get the MAC address associated with an IP address. The host broadcasts a request containing the node's IP address, and the node with that IP address replies with its MAC addres
Note: its message are directly encapsulated by a link layer protocol like MAC, and it is communicated within the boundaries of a single subnet and is never routed.


### ARP packet structure {#arp-packet-structure}

I use an image from wiki to illustrate the structure of ARP message.
![](/cs144/images/lab5_arp_structure.png)

-   Hardware Type ( HTYPE ):16 bits
    it specifys that which network link protocol type, use 1 indicates ethernet(MAC).
-   Protocol Type ( PTYPE ): 16 bits
    This field specifys the internet protocol for which the ARP request is intended. Like IPV4 the value of this field is 0x0800.
-   Hardware Length( HLEN ): 8 bits
    Length for Hardware type address, for ethernet is 6.
-   Protocol Length (PLEN): 8 bits
    Length for Protocol address, for ipv4 is 4.
-   Operation (OPER): 16 bits
    specifys the option of the sending, 1 for request 2 for reply.
-   Sender Hardware Address (SHA): 48bits
    Media address of the sender. In an ARP request this field is used to indicate the address of the host sending the request. In an ARP reply this field is used to indicate the address of the host that the request was looking for.
-   Sender Protocol Address (SPA): 32bits
    Internetwork address of the sender.
-   Target Hardware Addres (THA): 48bits
    Media address of the intended receiver. In an ARP request this field is ignored. In an ARP reply this field is used to indicate the address of the host that originated the ARP request.
-   Target Protocol Address(TPA): 32 bits
    Internetwork address of the intended reciever.


### Broadcast ARP message {#broadcast-arp-message}

If we don't the next-hop ethernet address, we need to send an ARP message to broadcast within the subnet. Here is the ARP message, assuming that the next-hop ip address is 192.168.0.1, our ethernet address is 48:32:12:98:78:90, our ip address is 4.3.2.1
I use an image to show the ARP message below.
![](/cs144/images/lab5_arp_message.png)
And the ARP message is within in etherframe, so the ARP message occupies the payload of the etherframe.
![](/cs144/images/lab5_ethernet_frame.png)
The src is 48:32:12:98:78:90, and dst is **ff:ff:ff:ff:ff:ff**, which indicates a broadcast message.After that, we can just send the message to the subnet


## network_interface {#network-interface}


### structure of class {#structure-of-class}

we use linked-list as our main structure to handle the mapping and no-nexthop ip dgram.

```c++
struct ip_mac_map
{
  // cache mapping IP->MAC
  uint32_t ip;
  EthernetAddress mac;
  size_t ms_tick;
};

struct dgram_no_mac
{
  InternetDatagram dgm;
  uint32_t next_hop;
  size_t ms_tick;
};
```

In this lab, each item of the list has a ms_tick to handle whether or not this mapping is valid and whether or not drop the dgram without nexthop ethernet address.
The function tick method is just to update the mapping in the list file, if some of each exceeds 30 secs we drop it out, but be careful of erase list when iterating. Also it has the ability to decide whether or not drop the dgram in the no-mac dgram. Actually we don't want to flood the network using ARP message repeatedly, so we need to check if the message we will send has the same ip as item in the no-mac linked list, and the item exists for less than 5 secs, we don't send our arp message again, just queue or drop. (I didn't handle that)


### why not queue? {#why-not-queue}

If i push staff for without no-mac of the nexthop, the next time arp message received, it might not be the first item pushed, and the queue can't take out the second but only the head. That's why I choose list to implent this.

Actually this lab is not hard, and there are also some cases to handle in the future.
![](/cs144/images/lab5_successfully.png)
