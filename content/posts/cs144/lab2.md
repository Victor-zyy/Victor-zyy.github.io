+++
title = "cs144-lab2"
author = ["zyy"]
date = 2025-03-22T20:11:00+08:00
tags = ["cs144", "c++"]
categories = ["net"]
draft = false
showtoc = true
+++

## Outline of TCPreceiver {#outline-of-tcpreceiver}

From the lab0, and lab1 we have implemented the ByteStream and Reassembler, in this lab2, I am gonna implement TCPreceiver.

First of all, we have to illustrate , in TCP (transmition control protocol), there may be many cases of the packets sent by the tcp peer of sender(versus the contrary). For example, there may be lost, altered, reordered, duplicated. And the TCP protocol was designed to handle these cases , so did I.


### reordered (out-of-order) or duplicated {#reordered--out-of-order--or-duplicated}

We can use **index(seqno)** to avoid this case, just like we analysed in lab1 the checking conditions of reassembler .
For example,

```c
String "abcdef"

push("ab")                 push("ab", index:0)
push("ab")                 push("ab", index:0)
push("abcd")   ----------->push("abcd", index:0)
push("f")                  push("f", index:5)
push("e")                  push("e", index:4)

```


### altered {#altered}

As we all know, the stack of service abstraction is like,

-   short gets(DNS, DHCP)-&gt;User datagrams-&gt;Internet Datagrams
-   Byte Stream(TCP) -&gt;User datagrams -&gt;Internet Datagrams
-   -------------Web requests/responses(HTTP)-&gt;Byte Stream
-   --------------------------Youtube/wikipedia-&gt; Web requests/responses
-   -------------Email sending(SMTP)-&gt;Byte Stream
-   -------------Email receiving(IMTP)-&gt;Byte Stream

In the OSI model or industrial network model, the TCP will use IP protocol to send message or receive, and the IP has a checksum for check-up when the packets receive, it will drop it off if the packets are not right enough.
So we the bytestream altered, the IP protocol will help us to drop it and handle this case.


### missing or lost {#missing-or-lost}

Like UDP, we can't retransmit the message over and over again until we got the answer. So we have to use another method to handle this case. **Acknolegement**.

```c

Byte String "A B C D E F G FIN" FIN indicates end of the string.
Sender:                                          Receiver:
"A index:0"
"B index:1"
"C index:2"
"D index:3"   missing in the trip
"E index:4"
"F index:5"
"G index:6"
"FIN index:7"                                    {ack:3}
we assume that, the network area of sender is fast than receiver.
That is why sender sends all messages before receiver receives.
```

The receiver gives ack 3 means that, index 0 1 2 are received in the stream, next we need index 3.
Note, we have to give FIN a number, when the ack 8 means we received FIN which can be an notion for the sender that it's time to close the connection.
However, is it enough? TCP is a flow-control protocol, the practical recv message is &lt;ackno: &lt;&gt;, window_size: &lt;&gt; &gt; which tells the sender the size of slide-window. In some situations, the window-size is far more meaningful.


## Implemention of Class {#implemention-of-class}

In this lab2, the outline of the TCP receiver is like,
![](/cs144/images/lab2_tcp_class_outline.png)
**Note**: rule of TCP reply to any nonempty message.(which i haven't noticed that until i implented the class of tcp receiver).


### end of stream {#end-of-stream}

And if the a sender has ended the stream, maybe the incoming bytes has not received yet. So the peer of the sender of TCP may not be closed. But at the same time, another connection has built used the same port number of the sender, and the last peer of sender will send the last packet, it will casue conflit with the new built sender.

To handle this we use a random big number as start of ISN (initial sequence number) + SYN to indicates the seqno, which can diminish the conflits. But in RFC of TCP header, the seqno is 32 bit, as the seqno grows it will wrap around to zero, so we have to do some arithemtics to handle this meric.

Note: the first seqno is to SYN, and the last seqno is FIN. So we should acknolege each of them.


### Three-way-handshake {#three-way-handshake}

```nil
without SYN and seqno:
- I: {{first_index=0, data=”abcdef”, FIN=true}, {next_needed=0,
window_size=1000}}
- Peer: {{first_index=0, data=””, FIN=true}, {next_needed=7,
window_size=1000}}
- I: {{first_index=7, data=””, FIN=false}, {next_needed=1,
window_size=1000}}

no end of the stream, and the peer mistakes that the sender need to bytes from it. the next_needed sometimes will misunderstand the peer I think.


with SYN:
- I: {{seqno=12340, SYN=true, data=”abcdef”, FIN=true}, {What should
this be? (before seeing 9999 from the Peer)}}
- Peer: {{seqno=9999, SYN=true, data=””, FIN=true}, {next_needed=12348,
window_size=1000}}
- I: {{next_needed=10001, window size =1000}}

That's why we need a state ack to inform the peer of direction of bytestream.

upgrade ACK <ACK flag, ackno>:
- I: {{seqno=12340, SYN=true, data=”abcdef”, FIN=true}, {ACK=false,
ackno={missing}, window_size=1000}} (SYN)
- Peer: {{seqno=9999, SYN=true, data=””, FIN=true}, {ACK=true,
ackno=12348, window_size=1000}} (SYN+ACK)
- I: {{ACK=true, ackno=10001, window size =1000}} (ACK)

This is three way handshake of TCP(SYN+SYN-ACK+ACK).
```


## Wrap and Unwrap {#wrap-and-unwrap}

From above, we know seqno is 32bit which will wrap around, for the UINT32_MAX is limited in network transmition. So we need to transfrom index from one to another.Here is the index table which represents the relationship between all of them.
![](/cs144/images/lab2_wrap.png)

Wrap implemention like this.

```c++
// ( SYN + n ) mod 2^32
return Wrap32 { zero_point + static_cast<uint32_t>(n)};
```

And Unwrap is kind of noying, that is because when given a number like 15, if the ISN is zero, the absolute index might be 15 , 15 + 2^32,  15 + 2^33, .etc. That's why we need another checkpoint index  closest near it to help find out the right index.

```c++
uint64_t Wrap32::unwrap( Wrap32 zero_point, uint64_t checkpoint ) const
{
  // SYN: 2^32 -2 this->raw_value_;
  // seqno 2
  // absolute seqno 2^32 + 4
  uint32_t off = this->raw_value_ - zero_point.raw_value_;
  if(this->operator==(zero_point)){
    // 0 , 2^32 , 2^33 .etc
    // [(checkpoint >> 32 << 32 ) , (checkpoint >> 32 + 1) << 32]
    // checkpoint >> 31
    bool last = checkpoint & 0x0000000080000000;
    return (1UL << 32) * ((checkpoint >> 32) + last);
  }else{
    if(checkpoint <= off)
      return off;
    bool last = ( checkpoint - off )& 0x0000000080000000;
    return (1UL << 32) * (((checkpoint - off) >> 32 ) + last)+ off;
  }
}
```

-   seqno equals to zero_point(ISN)

This condition is simple, because it is not 0, or 2^32, or multiples of 2^32 which is near to checkpoint. And the way to get the nearest index is to find checkpoint is above or below the middle of the two.

```nil
          |--------|------------------|
2^(32+n-1)    checkpoint             2^(32+n)
we use checkpoint>>31 maybe 0x11 that's above middle or 0x10, that's below middle. use 31 not 32 to indicates the above or below middle which is helpful.(I dbg the tests finding that connection).
```

-   seqno not equals to zero_point(ISN)
    we subtract an offset and still use the method above. And if you say off is minus zero, yes that's true, and the unsigned will wrap it to plus value. **Note**: the minimum is off if the checkpoint is less than off.


## TCP Receiver Class {#tcp-receiver-class}


### keep track of start {#keep-track-of-start}

I use a class private var to track it, when it is true to insert substrings into the reassembler.


### index calculate {#index-calculate}

absolute index and real payload index in assembler.

```c++
// Push any Data into the Reassembler
uint64_t seqno = message.seqno.unwrap(this->isn, this->reassembler().unassembled_index());
// insert index starts from zero and pointed by payload
this->reassembler_.insert(message.SYN + seqno - 1, message.payload, message.FIN);
```


### send message {#send-message}

Rule: TCP will reply any nonempty message. and we need to check whether the state_s is true or false to indicate the ack flag, and ackno is next_indexed(which is offered by reassembler ). RST state need to be designed when the bytestream has an error(the ByteStream has had this api provided).

```c++

TCPReceiverMessage TCPReceiver::send() const
{
  uint64_t win_size = this->writer().available_capacity();
  tcp_m.window_size = win_size >= UINT16_MAX ? UINT16_MAX : win_size; // to avoid overflow
  // unassembled_index + 1 (SYN must accept it) + FIN(may be not ending state so we use function of reassemble_connect_state to represent it)
  tcp_m.ackno = state_s ? ack.wrap(this->reassembler_.unassembled_index() + 1 + this->reassembler().reassemble_connect_state(), this->isn) : std::optional<Wrap32>{};
  tcp_m.RST = this->reader().has_error();
  return tcp_m;
}

```

Last but not least,  Window size is 16 bits, when the capacity of bytestream is up to or above this limit, we must do some check to avoid this overflow when assignment.

Finally,  successfully passed the tests.
![](/cs144/images/lab2_successfully_pass.png)
