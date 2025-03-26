+++
title = "cs144-lab4"
author = ["zyy"]
date = 2025-03-23T15:25:00+08:00
tags = ["cs144", "c++"]
categories = ["net"]
draft = false
showtoc = true
+++

## Glue of CS144TCPSocket {#glue-of-cs144tcpsocket}

I will illustrate the glue for using CS144TCPSocket in one image.
![](/cs144/images/lab4_cs144_framework.png)
From the picture, we know that it's our job to implement TCP/IP in user layer in this class. But how do our packets of network send to other computers through the internet ? The answer is to use tun/tap device under the linux kernel. The tun/tap device is kinda like a software net-card which tun device gives you raw IP packet and tap device gives you raw MAC dataframe. And follow the instructions of the tun.sh, setting the iptables to route makes our packet to go elsewhere.

Let's see what the tun.sh done in this lab.


### tun.sh {#tun-dot-sh}

The most important part of tun.sh is to create a tun/tap device and set the iproute tables. The snippet of the code is shown below.

```sh
TUN_IP_PREFIX=169.254
local TUNNUM="$1" TUNDEV="tun$1"
ip tuntap add mode tun user "${SUDO_USER}" name "${TUNDEV}"
ip addr add "${TUN_IP_PREFIX}.${TUNNUM}.1/24" dev "${TUNDEV}"
ip link set dev "${TUNDEV}" up
ip route change "${TUN_IP_PREFIX}.${TUNNUM}.0/24" dev "${TUNDEV}" rto_min 10ms

# Apply NAT (masquerading) only to traffic from CS144's network devices
iptables -t nat -A PREROUTING -s ${TUN_IP_PREFIX}.${TUNNUM}.0/24 -j CONNMARK --set-mark ${TUNNUM}
iptables -t nat -A POSTROUTING -j MASQUERADE -m connmark --mark ${TUNNUM}
echo 1 > /proc/sys/net/ipv4/ip_forward
```

```shell
ubuntu:~$./tun.sh start 144
```

And I substitue this command to make it clear to see. Here is the translated snippet.

```shell
ip tuntap add mode tun user zyy name tun144
ip addr add 169.254.144.1/24 dev tun144
ip link set dev tun144 up
ip route change 169.254.144.0/24 dev tun144 rto_min 10ms
# Apply NAT (masquerading) only to traffic from CS144's network devices
iptables -t nat -A PREROUTING -s 169.254.144.0/24 -j CONNMARK --set-mark 144
iptables -t nat -A POSTROUTING -j MASQUERADE -m connmark --mark 144
echo 1 > /proc/sys/net/ipv4/ip_forward
```

From the manual, we can bind an IP address to tun device. After binding, the network address is
169.254.144.0/24 which has 24bits subnet-mask. also, the broadcast address is 169.254.144.255/24
,169.254.144.1/24 host address
Note:default getway, set all host bit to 1 except the last bit.
169.254.144.254/24 default getway


### route table show {#route-table-show}

```sh
$ip route show

default via 10.198.255.254 dev wlp1s0 proto dhcp metric 600
10.198.0.0/16 dev wlp1s0 proto kernel scope link src 10.198.129.197 metric 600
169.254.0.0/16 dev virbr0 scope link metric 1000 linkdown
169.254.144.0/24 dev tun144 scope link linkdown rto_min lock 10ms
169.254.144.0/24 dev tun144 proto kernel scope link src 169.254.144.1 linkdown
169.254.145.0/24 dev tun145 scope link linkdown rto_min lock 10ms
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown

$route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.198.255.254  0.0.0.0         UG    600    0        0 wlp1s0
10.198.0.0      0.0.0.0         255.255.0.0     U     600    0        0 wlp1s0
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 virbr0
169.254.144.0   0.0.0.0         255.255.255.0   U     0      0        0 tun144
169.254.144.0   0.0.0.0         255.255.255.0   U     0      0        0 tun144
169.254.145.0   0.0.0.0         255.255.255.0   U     0      0        0 tun145
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0

```

From the iproute tables, we know 169.254.144.0/24 network address can be routed to elsewhere catching the default getway.


### NAT (Network Address Translation) {#nat--network-address-translation}

```sh
zyy@ubuntu:~$ sudo iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
CONNMARK   all  --  169.254.144.0/24     anywhere             CONNMARK set 0x90

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
LIBVIRT_PRT  all  --  anywhere             anywhere
MASQUERADE  all  --  anywhere             anywhere             connmark match  0x90

Chain LIBVIRT_PRT (1 references)
target     prot opt source               destination
RETURN     all  --  192.168.122.0/24     base-address.mcast.net/24
RETURN     all  --  192.168.122.0/24     255.255.255.255
MASQUERADE  tcp  --  192.168.122.0/24    !192.168.122.0/24     masq ports: 1024-65535
MASQUERADE  udp  --  192.168.122.0/24    !192.168.122.0/24     masq ports: 1024-65535
MASQUERADE  all  --  192.168.122.0/24    !192.168.122.0/24
```

In the CS144Lab4,  webget app binds an ip-address of 169.254.144.9, which belongs to tun144 ip network.

```c++
multiplexer_config.source = { "169.254.144.9", std::to_string( uint16_t( std::random_device()() ) ) };
```

As we can see the iptables above, let's illustrate the meaning of it by picture.
![](/cs144/images/lab4_ip_tables.png)
If the source is in 169.254.144.0/24 network address, it will be set as mark 0x90 using masquerade target to route to elsewhere.
And when the packet is coming, the kernel will choose to route to write application. That's the lab4 routing strategy.


## Analyse Data {#analyse-data}

I collected for about 5 hours of data from 20250312 9:00:00 to 20250312 14:19:00. And I write a python script file to analyse the data to answer the 10 questions.

-   question 1: echo_reply_rate

> loss               :     115
> dup                :     41
> echo_receive_rate  :     0.9987992440458167

-   question 2: longest concecutive

> max_consecutive    :     8309

-   question 3: longest missing concecutive

> max_missing        :     8

-   question 4: independent or correlated

> N+1_reply_N_missing:     0.0006690501578540217
> N+1_reply_N_reply  :     0.999330949842146

From the result, we can see that it's not idependent.

-   question 5/6: get minimum and maximum RTT

> min RTT            :     20.2
> max RTT            :     495

-   question 7: Make a graph of the RTT as a function of time. Label the x-axis with the actual time of day (covering the 2+-hour period), and the y-axis should be the number of milliseconds of RTT.

{{< figure src="/cs144/images/lab4_rtt_function.png" >}}

-   question8: Make a histogram or Cumulative Distribution Function of the distribution of RTTs observed. What rough shape is the distribution?

{{< figure src="/cs144/images/lab4_histogram.png" >}}

-   question9: make a graph of the correlation between “RTT of ping #N” and “RTT of ping #N+1”.

The x-axis should be the number of milliseconds from the first RTT, and the y-axis
should be the number of milliseconds from the second RTT. How correlated is the RTT
over time?
![](/cs144/images/lab4_correlation.png)
I see but not clear that if this time RTT is large, maybe the second is small.


## cs144 note {#cs144-note}

:: scope operator which I haven't noticed before
::close function when in function close

```c++
void FileDescriptor::FDWrapper::close()
{
  CheckSystemCall( "close", ::close( fd_ ) );
  eof_ = closed_ = true;
}
```
