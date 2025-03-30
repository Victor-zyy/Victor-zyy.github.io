+++
title = "cs144-lab7"
author = ["zyy"]
date = 2025-03-30T17:10:00+08:00
tags = ["c++", "cs144"]
categories = ["net"]
draft = false
showtoc = true
+++

## Object {#object}

The lab7 is the final lab of the cs144 network class, it doesn't need a single line of code to write. But the idea of this whole structure is really nice to follow, and I will do and analyse these structures later on, but today, i give you a short framework of the lab7 and achievements.


### outline {#outline}

The picture of this doc is nice to illustrate, lab7 uses some code to glues the bottom layer and the details things. It provided us an illusion that we use our components of code to connect and send files.
![](/cs144/images/lab7_structure.png)
It use cs144.keithw.org as a relay to communicate one with another. Then the host and server are likely to connect without hardware bariiers.


### error of code {#error-of-code}

what we do is compile and run the apps called endtoend to connect between host and server.Actually when I run my codes composited together, it will throw an error exception **optional access error**, which annoys me for a couple of hours to debug. Then I finally find that I access the value before check the std::optional value existence.


### three handshake {#three-handshake}

{SYN, SYN+ACK, SYN}
From the debug messages we can see the three handshake of TCP connection, from then on, I realize that in the past I have no idea of what happened behind the TCP, now I have known that it will use ARP to get MAC address and the router will route the message to the right destination of nexthop, which excited me a lot.


## Achievements {#achievements}


### chat room {#chat-room}

After debugging, I can use my codes of components to do lots of cool things.
![](/cs144/images/lab7_connection_ok.png)
And the quit is ok too!
![](/cs144/images/lab7_quit_ok.png)


### send file {#send-file}

{{< figure src="/cs144/images/lab7_successfully.png" >}}

Finally I benefited a lot from cs144, and it's worthwile to do. Although I haven't fully understand the structures of the labs, and I will do it later on.
