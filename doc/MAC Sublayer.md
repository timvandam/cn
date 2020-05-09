# MAC Sublayer
The MAC (*Medium Access Control*) sublayer is responsible for deciding who can use the communication channel, it resides
at the very bottom of the Data Link Layer.

## Index
- [The Channel Allocation Problem](#the-channel-allocation-problem)
- [CSMA/CD](#csmacd)
- [Protocols in Wireless Channels](#protocols-in-wireless-channels)
- [MAC in Classical Real-World Protocols](#mac-in-classical-real-world-protocols)

## The Channel Allocation Problem
Every cable is only capable of sending so much data per time unit; its bandwidth is limited. This limitation requires us
to properly divide the time each machine can use the cable. In other words, we have to allocate the cable between
different users. There are different ways of allocation.

### Static Allocation
Static allocation divides a channel into parts, where every user gets a part. This makes it possible for each user to
use the medium at the exact same time. The downside of this is that the amount of data that can be sent and received by
a user's partition of the medium is usually very small. Since the total transfer rate is evenly divided among users,
even if a user A is not using the medium, user B will still have the exact same data transfer rate as when user A *is*
using the medium. This might seem nice, but this definitely not nice as it not only wastes bandwidth but also causes
each partition of the medium to be extremely slow.

### Dynamic Allocation
Dynamic allocation uses all bandwidth available when needed. It gives users bandwidth whenever they need. It works based
on the assumptions that:
1. Traffic is independent - traffic from node A is unrelated to traffic from node B;
2. There is only one channel/link over which to communicate;
3. If two frames happen to collide, it can be detected;
4. Continuous or slotted time - you can send data whenever you want;
5. Nodes can detect whether the channel is in use.

---

The way channel allocation is implement highly depend on whether it's possible for users to detect whether the medium is
in use or not. This creates a difference between allocation *with* carrier sense and allocation *without* carrier sense.

### Allocation without Carrier Sense
The oldest protocol without carrier sense is *ALOHA*; users transmit frames whenever they want. If a collision were to
happen you would simply retry transmission after a random delay. You could use a protocol like ALOHA when your medium
doesn't have many concurrent users as it does not scale very well; every packet that overlaps with another packet will
cause both packets to fail, which is absolutely horrific for performance. 

Protocols to which you can send data at any time are a form of *Utopian Simplex*.

To improve performance without carrier sense it is possible to allocate time slots to users. This makes the system much
more complex as problems such as time synchronization have to be solved. This boosts performance by a factor of 2. The
downside of this approach is that not all bandwidth can be used (in case a user is idle).

### Allocation with Carrier Sense
Having carrier sense allows for a lot of magic to happen. Carrier Sense allocation is also called *Carrier-Sense Multiple
Access*, or **CSMA** for short. There are a few protocols that apply CSMA:
1. 1-persistent - wait for the channel to be free, then send;
2. Nonpersistent - if busy, wait for some random amount of time, try again;
3. `p`-persistent - if busy, wait for next slot. if idle, send with probability `p`.

A greedy protocol like 1-persistent is good for low load, but generous protocols perform better when under high load.

#### 1-persistent
A downside of 1-persistent is that collisions can occur; two users will both be told that the channel is free, and will
both send their data, which will then collide. When two messages collide, a *random backoff* will occur - the users will
stay quiet for some time and then start asking whether the channel is free again, etc.

1-persistent is not scalable at all, as collisions will happen the moment more than 1 user wants to send data.

#### Nonpersistent
Nonpersistant is very similar to 1-persistent. The only difference is that a user will wait for some time before asking
whether the channel is free after having been told it wasn't. This is effectively a random backoff. The downside of
nonpersistent CSMA is that it creates some extra latency and creates cases where no data is sent, while a user does have
data to send. In short, nonpersistent CSMA creates the opposite problem that 1-persistent creates; users are not greedy
enough.

#### `p`-persistent
`p`-persistent CSMA attempts to compromise between 1-persistent and nonpersistent. It uses slotted time, but instead of
sending data whenever a user's time slot opens, it will only does this with a certain probability `p`. `p` determines
how aggressive the protocol is; the lower `p` is, the lower the amount of collisions, the higher `p` is, the higher the
amount of collisions. In case of a collision a random backup will occur.

## CSMA/CD
*CSMA/CD* is an improvement on *CSMA*. The 'CD' stands for *Collision Detection*, but this is not like normal collision
detection; instead it can detect collisions before they happen at all. This allows a user to stop sending their message
in case of potential collisions, preventing them altogether. This not only saves time but also bandwidth. Users will
transmit a single bit during the **contention period**, whoever's bit arrives first (without collisions) will be the
winner and will be able to transmit their data, which starts the **transmission period**. Note that collisions still
happen, but they will only happen during the contention period, in which only singular bits are sent, making this a
cheap process. It can take as long as twice the time to propagate the Ethernet to detect collisions.

Here are a few protocols that implement CSMA/CD:

### Basic Bit-Map
With basic bit-maps, every user will have their own contention slot. Users of the medium can then transmit a 1-bit at
the exact time of their contention slot. Since all users are on the same BUS, all users will know of this. This allows
them to coordinate the order in which they will send their frames. A disadvantage of this system is that it does not
scale particularly well; when there are many users the contention period will only get longer; 1000 devices create 1000
contention slots, etc.

### Token Ring
Token Rings are quite simple; all users send their messages in a predefined order. Whenever a user has the *token*, they
can transmit their data. Once they're done transmitting their data (or don't have data to transmit) the token will be
passed on to the next user. The order of users is pre-defined based on e.g. their WiFi number. An obvious downside of
this is that whenever users join and leave the ring needs to be reconstructed, so proper management is a must. 

### Binary Countdown
Binary Countdown is an improvement on the Basic Bit-Map. Each user will have their own station number, which will gain
some priority for every 1-bit in their number. The higher their number, the more priority they have. Each bit-time they
will send one bit of their station number, so whenever a 1-bit is present, all 0-bit stations will have lost and stop
trying to gain access to the channel. Once there is just one station left, that station will be allowed to transmit.
This is much more scalable than basic bitmaps as not all users will have their own bit. Instead, they just transmit to
enter the race for channel access only whenever they want to send data. The overhead is now `log(n)` instead of `n` for
basic bit-maps.

## Protocols in Wireless Channels
Wireless channels are very convenient, but make it harder to detect collisions.

### Hidden & Exposed Terminals
Picture the following scenario: three phones want to communicate with each other. Phones A & B, and B & C are within
range of each other, but A & C are not. In this case A and C cannot communicate directly and will not know whether they
are transmitting or not. This means that if both A and C want to send a message to B, they won't know if B is already
receiving data; carrier sense fails, and collisions are bound to happen. When collisions *do* happen, A and C will also
be unable to detect them. A and C are each other's hidden terminals, while B is an exposed terminal for both A and C.

When B wants to send data to A, C will sense that B is busy, thus C will not transmit data. However, even if C did
transmit data no collisions would be caused. So exposed in this situation exposed terminals will cause unnecessary
delays.

#### MACA 
The solution to the previously described problems is *MACA* (Multi Access with Collision Avoidance). With MACA, whenever
a node wants to send data to another, it will first send a **Request To Send**. All exposed nodes will receive this,
and the receiver node will respond with a **Clearance To Send**, while other exposed nodes will wait. The exposed nodes
that don't receive a CTS are outside of the range of the receiver node, so it will be safe for them to transmit data to
other available nodes (which will be hidden nodes to the original receiver node). MACA is a very simple but effective
solution to the described problem.

## MAC in Classical Real-World Protocols
Machines used to share a single Ethernet connection, which meant that CSMA/CD was needed to transmit data succesfully.
In classes Ethernet 1-persistent CSMA/CD was used, where **BEB** (Binary Exponential Backoff) was used to compute the
random backoff time. With BEB, the number of slots to wait depends on the amount of failed attempts, using the formula
2<sup>i</sup> - 1. This technique would be efficient for large frames, but not for smaller frames. This is because
bigger frames mean less contention period in the same time frame. This shows that the contention period can be quite a
bottleneck for small messages.

### Classic Ethernet Frames
|Ethernet (DIX)|Preamble|Destination Address|Source Address|Type|Data|Pad|Chunksum|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Bytes|8|6|6|2|0-1500|0-46|4|

|IEEE 802.3|Preamble + S o F|Destination Address|Source Address|Length|Data|Pad|Chunksum|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Bytes|8|6|6|2|0-1500|0-46|4|

### MAC in 802.11 (WiFi)
Wireless stations can not detect collisions. Hence, they rely on ACKs to determine whether collisions occurred. If an ACK
is lost the sending will simply assume that a frame was lost, resending it. These stations *can* use MACA, but usually
don't. Instead of detecting collisions, they are avoided: **CSMA/CA**.

#### CSMA/CA
Some core elements of CSMA/CA are:
1. Physical Channel Sensing
    - Sensing whether the physical channel is in use.
    - If it is, wait for the channel to be idle.
2. Virtual Channel Sending
    - Frames carry a **Network Allocation Field** (NAV) that indicates the transmission's length. This allows stations to
    wait for the end of a transmission.
