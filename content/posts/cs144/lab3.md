+++
title = "cs144-lab3"
author = ["zyy"]
date = 2025-03-22T20:17:00+08:00
tags = ["cs144", "c++"]
categories = ["net"]
draft = true
showtoc = true
+++

## TCP Sender Class {#tcp-sender-class}

TCP  Sender Responsibility,

• Keep track of the receiver’s window (receiving incoming TCPReceiverMessages with
their acknos and window sizes)
• Fill the window when possible, by reading from the ByteStream, creating new TCP
segments (including SYN and FIN flags if needed), and sending them. The sender should
keep sending segments until either the window is full or the outbound ByteStream has
nothing more to send. **while loop**
• Keep track of which segments have been sent but not yet acknowledged by the receiver—
we call these “outstanding” segments . **outstanding segments**
• Re-send outstanding segments if enough time passes since they were sent, and they
haven’t been acknowledged yet. **timer class design**

Notes to be reminded of.

1.  when started, we see window size of the peer is 1, so we can just send SYN bit.
2.  tick method, when it was called, it proves that how many time passed, we don't have to use clock or time function
3.  when received a ackno, how to compare (Wrap32)

Basic implementation of tcp sender class


### push method {#push-method}

first , we need to keep track of the window size of the TCPreceivermessage, use member variable. The basic idea of the method is to fill the window as possible. In here, we need to judge the ByteStream bytes_buffered , TCPconfig:MAX_PAYLOAD_SIZE, and the window size. Note: the TCPconfig:MAX_PAYLOAD_SIZE is only used for payload.

When there is no message received at all, we just need to send SYN bit only. And if the message is comming, we need to fill the window as soon as possible. After that, add FIN bit in the end. The FIN bit is sometimes nusty when i implement this function. I use a close_state to indicate the FIN state, if the FIN state is sent, we see the connection is closed.

When the window size is zero, we need to transmit one bit to the peer.

I use list to keep track of the outstanding segments, when each segment is sent, we add it into the list. Using list, we can iterate easily.


### receive method {#receive-method}

Each time , we received a TCPReceivermessage, we have to compare the ackno (absolute seqno greater than). But the Wrap32 will wrap around sometimes, so we just use operator== to compare. In the list, operator== matches it will return the iterator, then we use erase to remove it. Actually we have to do some arithematic to solve it, but it doesn't matter.

Besides, we are supposed to update the this-&gt;recv_message_ which tracks the window size and wind_zero attribute. It's quite important, so we have to do it.


### make_empty_message {#make-empty-message}

Actually i don't fully know why to implement this function, but What i know is that set the right seqno will make it easy. So we need an another var to keep track of the bytes_len transmitted.


### tick {#tick}

The tick function is really nusty for me, at first, we have the RTO initial value, when the tick is called , it says that how many times passed, if the time expired which means the passed time is greater than initial RTO, we have to resend the lowest seqno, and check the window_size to decide whether we should double the RTO or not, last but not least add the consecutive_transmissions for one.

I implemented a timer class to do it, when to start the timer, and when to reset. Each time we push a valid segment we need to start the timer. And each time we recv an ackno, we should decide whether to reset or not.


## Bug Advice {#bug-advice}

when use STL library, check the empty first will minus the time you debug. When a ADDRESS_SNIFFER came out in the debug, you should consider if we forget checking the STL library before operating the container.


## Corner Cases {#corner-cases}

There are many corner cases to remind of, but i don't wanna metion in this blog post.  Finally after great efforts, I finally succeeded.

{{< figure src="/cs144/images/lab3_successfully.png" >}}
