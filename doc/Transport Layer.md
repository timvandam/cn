# Transport Layer
The transport layer links traffic to its application. It does this by using *ports*, which *sockets* can connect to to
communicate.

## Index
- [Addressing](#addressing)
- [Segment Headers](#segment-headers)
- [Sequence Numbers](#sequence-number)
- [TCP Message Types](#message-types)
- [Setting up a TCP Connection](#setting-up-a-tcp-connection)
- [TCP ACKs](#tcp-acks)
- [Fast Retransmission](#fast-retransmission)
- [Initial Sequence Numbers](#initial-sequence-numbers)
- [Crash Recovery](#crash-recovery)
- [Flow Control](#flow-control)
- [TCP Congestion Control](#tcp-congestion-control)
- [Connection Release](#connection-release)

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

## Sequence Numbers
Sequence numbers in TCP are utilized for several reasons. While the data link layer already deals with quite a few
issues by using SEQ numbers (using sliding windows), not all of these can be solved on the data link layer.

Of the following issues, only the first one can be solved on the data link layer, the rest is done on the transport
layer:
1. Detect missing segments;
2. Detect whether segments arrive out of order;
3. Detect if parts of segments are missing;
4. Detect duplicates.

Note that the data link layer *does* detect when *frames* arrive out of order, are missing, or are duplicate. Segments
are made out of multiple frames, so entire segments are **not** covered by the data link layer.

## TCP Message Types
TCP headers have a few flags that can indicate the type of message. These are **SYN**, **ACK**, and **FIN**.

SYN is for establishing connections, ACK is for acknowledgements, and FIN is for closing/terminating connections. When
one of these message types are used, the message won't have a body.

---

TCP applies SEQ numbers in the following way:

Before starting a connection (SYN), the initializer will choose an initial SEQ. When receiving a SYN, you will also
decide an initial SEQ. The SEQ you locally store is your initial SEQ + the amount of bytes previously sent.

ACKs are the opposite; they're the initial SEQ of your partner + the number of received bytes. It's pretty
straightforward.

Since there are only 32 bits to represent SEQs/ACks, you should take % 2<sup>32</sup> of your SEQ/ACK numbers (to wrap
around).

In short: your next SEQ should be equal to the ACK of your partner (assuming everything tranmitted has arrived). Both
you and your partner pick an initial SEQ. Every byte you send adds 1 to your SEQ (SYN and FIN also do this, but ACKs
don't).

When received SEQs/ACKs don't match the locally stored ones you do trigger a retransmission (your partner retransmits).

## Setting up a TCP Connection
TCP uses a three-way handshake to set up connection. It works as follows:
1. Start handshake by locally choosing a SEQ `x`;
2. Send a SYN with SEQ `x` (new SEQ is now `x + 1` as SYN has byte length 1);
3. Partner will choose a SEQ `y`, and sets ACK to `x + 1`;
4. Partner will transmit a SYN-ACK with SEQ `y` and ACK `x + 1`;
5. Partner increments SEQ by 1 (`y + 1`);
6. You set ACK to `y + 1` (as SYN-ACK has byte length 1).
7. You send an ACK `y + 1` SEQ `x + 1`. SEQ/ACK stay the same as ACKs don't increment SEQs.

This is the end of the three-way handshake. As you can see, it consists of three messages. The internet probably has
some nice diagrams showcasing this. You can look those up yourself.

## TCP ACKs
There are two ways of doing ACKs with TCP:
1. Direct ACK: send a TCP header with the ACK field set to 1;
2. Piggybacked ACK: Increase ACK number in the next segment with an actual payload.
    - ACK field is set to 0, but ACK *is* incremented (after receiving the segment).
    
## Fast Retransmission
TCP uses a sliding window protocol, so it uses timers to detect when packets have been lost (unacknowledged). This is not
the only way to detect lost packets, though. For example, when the sender's SEQ doesn't match the receiver's ACK the
receiver can take action. This could also mean that packets are simply received out-of-order though, so immediately
triggering a retransmission might be a bit too aggressive. Instead, the third acknowledgement with the same SEQ number
will trigger a retransmission (given that there's an unacknowledged transition).

## Initial Sequence Numbers
It might seem obvious to just choose 0 as your initial SEQ, but this is not ideal. Picture this: two parties are
connected and both use initial SEQ 0. If party 1 sends a message, but immediately crashes after transmission, and
then reconnects before the message was delivered, party 2 will think it has just received a message on that connection.
This is obviously very bad, as party 2 will increment its ACK, while party 1 is still on SEQ 1 (just after connecting).

So the best initial SEQ is a SEQ that is unlikely to have been next in line on a previous connection.

### Clock-Based Sequence Numbers
Such an initial SEQ can be generated using *clock-based sequence numbers*, with which you take the 32 least significant
bits of the clock time to get your initial SEQ. This means that previously sent messages will be way behind the initial
SEQ (unless your SEQ happens to loop back to 0 due to the % 2<sup>32</sup>). Luckily, this can be solved! The solution
is to not use some numbers as SEQ. These unusable numbers form *forbidden regions*, which are numbers that will wrap
back to 0 in less than time `T`. This `T` can for example be the time for a packet to be delivered.

When your initial SEQ is in a forbidden region, *resynchronization of sequence numbers* occurs. The key is that once
you get close to the forbidden region you run a protocol that helps the two parties to select new SEQ numbers to
continue from. They are pretty much re-picking initial SEQs, so it behaves as if you reconnected.\

### Randomized Initial Sequence Numbers
Modern TCP versions don't use clock-based sequence numbers. This is because they're predictable, making it easy to
hijack TCP connections. Instead, they use *Randomized Initial Sequence Numbers*. You still have the same forbidden
region problem, which can be solved in the same manner (resynchronization).

## Crash Recovery
There are two different approaches to receiving data which require different ways of recovering from crashes.

The first approach is passing on incoming messages to layers above you, and once that's done, acknowledging the received
message. When a higher layer happens to crash your system, this means that that incoming message will remain
unacknowledged, causing the sender to retransmit the message. If you just recovered from that previous crash, this might
cause another crash. This is not ideal.

The other approach is acknowledging incoming messages the moment they arrive, and then passing on the data to higher
layers. When the receiver machine crashes *after* acknowledging, the data is lost as it will not be retransmitted by
the sender. That's not great, and can cause even more errors.

Unfortunately there is no way to solve this is on the layer that receiver the message (layer `k`). So we need to involve
the next layer (layer `k + 1`). So on the transport layer the application layer has to handle these errors.

## Flow Control
Flow control is not the same thing as congestion control; flow control depends on the capacity of the receiver while
congestion control depends on the capacity of the medium.

The TCP header includes a field called *window size*, which is used in flow control. The window size tells the sender
how much data the receiver can handle. It's an indication of how much space is left in your local window, so it pretty
much tells the sender that they can not send more than that amount of data, as that would not fit in the window. The
sender can then do nothing but simply wait until the receiver has some window space, which they will indicate by
repeating the last ACK, but with a different window size.

### Silly-Window Syndrome
When a receiver has a very small window, you get the *silly window syndrome*, where the sender constantly has to wait
for two acknowledgements before being allowed to send again. This causes long delays, which is only amplified if there
is already some latency. Since the data in the window is already being processed, and the sender can only transmit more
data once they receive an ACK with a positive window size, there is always a delay between having processed data, and
receiving more data to process. Greater latencies now cause a reduction of throughput.

## TCP Congestion Control
Both the network layer and transport layer are responsible for congestion control. The network layer sends signals when
there is congestion, causing re-routing, dropping packets, etc., while the transport layer is responsible for the amount
of network that is added to the network. This means that the transport layer should pause when it sees a network layer
signal indicating a congestion network. Don't confuse this with the network layer dropping packets - the transport layer
is completely unaware of this and will have to wait for a NACK of some sort (like an expired timer) before knowing that
a dropped packet never arrived. Making the transport layer aware of congestion completely eliminates this issue,
decreasing latency in case of congestion.

UDP doesn't care about congestion, so when all you use is applications that use UDP you might get some huge delays in
case of congestion.

There are a few unknown variables that could help control congestion:
- Available bandwidth;
- Network topology;
- Other clients.

### Dynamically Adjusting Bandwidth using Trial & Error
One way of dealing with not knowing the available bandwidth is by simply increasing bandwidth until you receive a
congestion signal, and then slowing it down again.

The disadvantage of this is that congestion doesn't just depend on you. For all you know your neighbor is downloading
Shrek in 8K, taking up all your shared fibre optic cable's bandwidth while you're not even close to your allocated
maximum bandwidth and still receive a congestion signal, so in some cases you might not be able to use your bandwidth.
This is not a huge problem though, as shortly after TCP will simply start increasing your bandwidth again.

### Congestion Detection
There are a few ways to detect congestion:
1. Explicit feedback from routers;
    - Routers can tell your computer to slow it down, but this is quite rare.
2. Loss;
    - Detect when your packets are being dropped and slow it down. This is the most common approach.
    - Can create false-positives when your timer expires before receiving an ACK, or when error codes doesn't match. 
3. Latency.
    - Detect when the time between receiving ACKs is bigger than the time between sent segments.
    - Takes a bit longer to detect congestion, but gives a better indication of transmission rate.

### Regulating Sending Rate
When multiple users are using bandwidth, you want to give them all an equal (fair) share of bandwidth. Since the amount
of bandwidth is usually unknown, we have to use a different approach to figure out how much we should allocate to each
user.

A few different approaches are:
1. Additive (rate + `a`, rate - `a`);
2. Multiplicative (rate * `m`, rate / `m`);
3. Combination of both.
    1. Additive increase/decrease;
    2. Additive increase, multiplicative decrease;
    3. Multiplicative increase, additive decrease;
    4. Multiplicative increase, multiplicative decrease.
    
The downside of using an approach that is strictly additive/multiplicative approach is that you never get an optimal
result; for a strictly additive approach, if you're above the efficiency line your bandwidth will decrease by a set
amount, then it will automatically increase, and once again receive a congestion packet making it go down again over and
over again. For strictly multiplicative approaches the exact same thing happens (but instead of adding a set amount of
bandwidth, the bandwidth increases/decreases by a factor `m`).

A better solution is an additive increase with multiplicative decrease. The amount of bandwidth will still jump around
the efficiency line, but will creep closer to the optimal point.

### Congestion Window
TCP applies this using a **congestion window**. The sender locally tracks this window and specifies how many segments
can be transmitted. While the previously discussed window in flow control makes sure clients can handle traffic, this
window makes sure the network can handle it. Both windows can pretty much VETO whether you're able to send; whenever
either window is too small you won't transmit data. This prevents both network and client overloads.

Whenever you receive ACKs, you can increase the congestion window (meaning you can send more). This limits the sending
rate to ensure that all links can deal with the amount of data being transferred. The ACK rate will thus of course be
the rate the slowest link can handle.

### TCP Slow Start
Choosing an initial congestion window is not handy, as the size of such a window depends on the network, and before
sending any data you will not know the specifications of that network. Instead, TCP slow start is used, where you start
with a congestion window of 1, and then double it with each received ACK until receiving a congestion packet. The
advantage of this approach is that you will quickly be transferring at maximum bandwidth (exponential). There are
multiple methods you can use to determine when to stop using slow start:
1. When an arbitrary threshold is met (depends on TCP version);
2. When there are missing ACKs (packet loss).

In the former approach, you would switch to additive increase.

This is obviously not slow, but is slower than the previous algorithm, where you would start with the flow control
window. Some sockets have huge buffers, so you would start with an equally huge congestion window.

### Slow Start in TCP Tahoe
TCP Tahoe is the first version of TCP. In TCP Tahoe, slow start will stop when a threshold is met. However, whenever you
lose a packet, the congestion window will be reset to 1, and the threshold is lowered.

The problem with this approach is that just one lost packet immediately slows down your transmission rate. Hence, TCP
Fast Recovery was introduced (TCP Tahoe + Fast Recovery = TCP Reno).

### TCP Fast Recovery
When a packet is lost in TCP Reno, instead of resetting the congestion window back to 1, it is reduced by a certain
factor (multiplicative decrease).

Newer TCP versions use different variations of the same key idea of decreasing congestion window and threshold (e.g.
using implicit signals to detect congestion, like packet loss, instead of explicit congestion signals).

Every OS usually uses a different variation of congestion control; Windows, for instance, uses packet less and e2e delay
to detect congestion, while Linux and MacOS only use packet loss.

## TCP Connection Release
Releasing TCP connections is not as trivial as it sounds. There are generally two approaches to this:
1. Asymmetric disconnect;
    - Disconnect without discussing with the other party.
2. Symmetric disconnect.
    - Disconnect after an agreement with the other party.
    
Asymmetric disconnect is easy, but can lead to trouble as the other party won't instantly know that you have
disconnected and will continue sending data, resulting in data loss. For example: the other party sends data `D`, you
send a disconnect message (or don't) stop listening, now you receive `D`. However, you're no longer listening at the
port at which `D` arrived, so will lose data. For this reason asymmetric disconnect is not a viable solution for TCP.
Hence, it uses something more closely related to symmetric disconnect.

With symmetric connection releases both participants agree to end the connection. Simple right? NOT!

### The Two Armies Problem
The two armies problem makes symmetric connection releases difficult. Without going into detail, the problem is that
you can never be sure that the other party has received the message. You might think that ACKs will solve this, but this
will let the previous sender know their message arrived, but the person that sent the ACK won't know whether their ACK
has arrived. There is no solution for this. Sending an ACK for an ACK would require that ACK to be acknowledged too,
with another ACK, which would also need an ACK... You get the gist.

Symmetric release uses FIN packets to disconnect. The party that wants to disconnect send first sends a FIN, the other
party then agrees by sending a FIN themselves, which is then acknowledged by the other participant. If your FIN goes
unanswered, or you don't receive a final ACK, you just do an asymmetric disconnect after a while.

So, it looks like this:
|A->B FIN|
|:---:|
|**B->A FIN**|
|**A->B ACK**|

All the main TCP header fields have now been covered! (except urgent pointer + options).
