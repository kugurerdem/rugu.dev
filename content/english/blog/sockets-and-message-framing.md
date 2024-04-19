---
title: 'Sockets and Message Framing'
archive: true
draft: false
date: '2023-11-21'
tags:
    - technology
---

I've recently been involved in a fintech project that demands high performance,
posing various challenges related to a solid understanding of low-level
concepts, concepts that are primarily relevant to the inner workings of the
tools and protocols used beneath the surface.


One challenge involved separating two tasks into different processes:

1) The main process, responsible for constructing the necessary business state from
incoming messages through a specific socket connection.

2) The monitoring process, allowing users to track relevant changes in the state.

To enable communication between these processes, I implemented Inter-Process
Communication (IPC), and sockets naturally came to mind as a suitable solution.

Since not all incoming protocol socket messages to the main process were
relevant to user interaction, I set up a mechanism to filter and process the
relevant messages for the front-end perspective. These refined messages were
then sent to the monitoring processes through a socket connection.

However, acknowledging that sockets operate at a low level and suspecting I might
be overlooking something about the socket protocol, I aimed to confirm that each
piece of data sent from the main process would result in one received data on
each monitoring process. Delving into the self-assigned task of understanding
how sockets work, it turned out that one sent message does not always have to
correspond with one received data.

When sending messages through a TCP socket, it separates our messages into
multiple parts called packets and sends those. The receiving socket then
reorders and reassembles those packets into a chunk of the original message which is
sent – a data – and appends it to the socket's buffer. Since it is said that
these packets are reordered and reassembled by the TCP socket itself, most
people assume that one sent data will result in one data received. The thing is,
sockets do not wait for all incoming packets to be reordered and reassembled to
append them to their buffer. Instead, they append the chunks that are validated
and arrive in the correct order as they keep coming. A socket can essentially be
considered a Duplex Stream; it does not frame your messages and does not
necessarily know which parts of the incoming/outgoing data actually constitute a
meaningful message from your application layer's perspective.

From the perspective of socket users, the inner workings of "packets" are not
something to be worried about. The chunks added to the socket's buffer are
simply referred to as "received data" or "data chunks". The meaningful portions
of this data, converging as intended by the sender to convey a message, are
simply labeled as "messages".

Since the sockets lack awareness of the types of data and the desired format for
transmission, users of the socket must devise methods to identify which portions
of the incoming data constitute meaningful messages. This process, commonly
known as message framing, involves two approaches: the length prefix approach,
where each message is prefixed by the byte length of its content, and the
delimiter approach, where messages are separated by a designated delimiter
character or sequence of characters.

What struck me the most about diving into these concepts was observing the
widespread tendency for people to misunderstand the behavior of one of the most
fundamental tools a developer may need to interact with – a socket. For
instance, consider the experience of a developer who highlights this issue:

> True story: I once worked for a company that developed custom client/server
> software. The original communications code had made this common mistake.
> However, they were all on dedicated networks with high-end hardware, so the
> underlying problem only happened very rarely. When it did, the operators would
> just chalk it up to “that buggy Windows OS” or “another network glitch” and
> reboot. One of my tasks at this company was to change the communication to
> include a lot more information; of course, this caused the problem to manifest
> regularly, and the entire application protocol had to be changed to fix it. The
> truly amazing thing is that this software had been used in countless 24x7
> automation systems for 20 years; it was fundamentally broken and no one
> noticed. [1]

This entire process once again shows the importance of us developers being
genuinely curious about the tools and protocols we work with. If I hadn't asked
such questions about the tool which I am using, just because the program works,
I might have fallen into the same pitfalls that others have encountered.

# References

1 - Cleary, Stephen. “Message Framing.” Message framing. Accessed November 21,
   2023. https://blog.stephencleary.com/2009/04/message-framing.html.
