# Transport Layer
The transport layer links traffic to its application. It does this by using *ports*, which *sockets* can connect to to
communicate.

## Index
- [Addressing](#addressing)
- [Segment Headers](#segment-headers)

## Addressing
In its simplest form, the transport layer links ports to IP addresses. I.e., when the network layer receives a message,
it is received at a certain IP address. The transport layer then redirects it to a certain port (which is specified in
the message).

This is done using some primitive techniques;
1. Listen - wait for another process to contact us;
2. Connect - connect to a process that is listening;
3. Send - send data over an established connection;
4. Receive - receive data over the established connection;
5. Disconnect - release the established connection.

*Listen*, *connect* and *disconnect* are used for managing connections. The actual data transmission takes place in *send* and
*receive*. Data transmission is done in *segments* (packet + transmission data like ports, error correction, sequence
numbers, etc.).

### Berkeley Socket Primitives
The mentioned primitives are usually extended in programming. These extended primitives are the *Berkeley Socket
Primitives*. These additional primitives are:
1. Socket - an object that creates a new communication endpoint;
2. Bind - assign a local address to the socket;
3. Accept - passively accept an incoming connection request.

The *accept* primitive comes in handy because it makes it possible to refuse connections when certain conditions are met.
The *socket* and *bind* primitives are quite self-explanatory; sockets are used to connect/listen, and each socket has a
local address bound to itself (usually meaning a port).

### Process Servers
Generally each application that communicates with stuff has its own socket, which binds to a certain port. When these
applications don't actually use these ports very often, you're pretty much just draining your battery for no reason.
This can be solved using process servers, which is a server that listens to many ports. When it receives a message and
can identify it, it will then start a server that can actually handle that message, and redirect it there.

An example could be when you receive an email - the process server might be able to identify a message as an incoming
email by its header. The process server will then boot up your mail server and send the message there. Once your mail
server is done it will simply shut down again. This way you only need one server to listen at all ports.

Unfortunately process servers are not always a solution; it's essential to be able to identify messages. This can
usually be done by checking the port over which the message was sent, however, when you run your Minecraft server at
port 110, all incoming messages will be identified as POP3 messages (emails).

So the problem is that there needs to be a known mapping of ports. Process servers are also not always that handy; when
you frequently receive emails it is not worth it to constantly start and stop the mail server. You could simply leave it
running!

### Multiplexing
Multiplexing makes it possible to identify messages.

An example could be sending incoming messages over one connection to certain ports. One stream of data is divided into
multiple streams (which are the ports).

The other way around is also possible; an application might use multiple connections over one port, in which you receive
two data streams, which are then combined into one data stream (which is the application's port). This case is not very
common though, as using multiple connections means having multiple IP addresses.

## Segment Headers
On the transport layer, data is sent in segments which consist of a header and a body. Segments are pretty much
equivalent to *packets* on the network layer and *frames* on the data link layer.

Segment headers contain data about the segment, and is usually divided into fields with a set size. Segment headers can
differ between transport protocols.

The UDP protocol, for instance, uses a header that contains source/destination ports, the body length, and a checksum
(making it a very thin layer on top of IP). The reason for not including an IP address in the header is because this is
handled by the network layer, not the transport layer. That is how these layers work after all; segments are located in
the body of packets (or divided over multiple packets).

UDP is not meant for reliability, and does do automatic transmission. You can trigger retransmissions if you notice an
incorrect checksum, though.

TCP, on the other hand, uses much more sophisticated headers. TCP's purpose is to provide a reliable end-to-end byte
stream. This means:
- Error Detection;
- Automatic Retransmission;
- Flow Control;
- Congestion Control.

The TCP header is much longer, including source/destination ports, SEQ/ACK numbers, TCP header length, message type
indicators, a checksum, an urgency pointer, window size (for flow control), and an arbitrary amount of 32-bit
options. The SEQ number happens to include the segment length.

What about congestion control, though? TCP is connection oriented, meaning hosts locally keep track of state information
related to the connection. This local state consists of:
- The next SEQ number to use;
- The next ACK number to use;
- Congestion window;
- Timers for retransmissions.

### Checksums
Both the TCP and UDP segment headers include a checksum. The data link layer also utilizes error detection, so why even
use error detection on the transport layer? The reason it that transmission errors don't only occur during transmission.
Routers, for instance, can drop packets or flip bits. Another reason is that the data link layer cannot capture every
error. The additional checksum on the transport layer could catch some of these. However, it's still possible for errors
to go undetected, but this is quite unlikely.

TCP and UDP both use a 16 bit ones' complement checksum. The sender computes the checksum like this:
1. Divide the header (excl. checksum) + body into 16-bit sequences;
2. Compute XOR over all of these sequences;
3. Take the ones' complement of the result (essentially flip all bits).

The receiver checks the checksum like this:
1. Divide the header (incl. checksum) + body into 16-bit sequences;
2. Compute XOR over all of these sequences;
3. If not all bits are 1, there was an error.

This method of computing the checksum is not perfect; if two bits at the same position in their respective 16-bit
sequences were to flip, these errors would go undetected.