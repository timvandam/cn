# MAC Sublayer
The MAC (*Medium Access Control*) sublayer is responsible for deciding who can use the communication channel, it resides
at the very bottom of the Data Link Layer.

## Index
- [The Channel Allocation Problem](#the-channel-allocation-problem)
- [CSMA/CD](#csmacd)
- [Protocols in Wireless Channels](#protocols-in-wireless-channels)
- [MAC in Classical Real-World Protocols](#mac-in-classical-real-world-protocols)
- [Data Link Layer Switching](#data-link-layer-switching)

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

CSMA/CA inserts backoff slots to avoid collisions, MAC uses ACKs/retransmissions for wireless errors.

#### 802.11 modes
There are two modes in 802.11:
1. Infrastructure Mode
    - There is one base station that decides who controls the entire thing - it controls who can talk, etc. This is what
    most networks use (and what pertty much all users can use)
2. Ad-hoc Mode
    - There is no centralized station - you can talk to anyone within your range. Every user is always listening making
    this mode expensive battery wise (hence phones pretty much never do this). This mode is much harder to take down as
    every part of such a network has control.

#### 802.11 Frames
|Frame|Frame Control|Duration|Address 1 (recipient)|Address 2 (transmitter)|Address 3|Sequence|Data|Check Sequence|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Bytes|2|2|6|6|6|2|0-2312|4|

Address 3 is for nodes that forward/relay messages (e.g. the base station, an ad-hoc node, etc.).

The frame control looks like this:

|Frame Control|Version = 00|Type = 10|Subtype = 0000|To DS|From DS|More Frag.|Retry|Pwr. Mgt.|Mode Data|Protected|Order|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Bits|2|2|4|1|1|1|1|1|1|1|1|

## Data Link Layer Switching
If you have many local networks within say an organisation, it is often very handle to join these to makeone big LAN.
This can be done with **bridges**. The opposite is also possible; you can split one LAN into seperate LANs using
**VLANs** (virtual LANs).

### Use of Bridges
There are a few reasons for using bridges:
1. The need to connect computers in separate LANs.
2. One big LAN over a big area is not feasible.
    - Ethernet cables can only get that long.
3. One big LAN might have to be split up into smaller LANs to accommodate for big loads.
    - Each LAN can only have a limited amount of bandwidth.

Bridges should be transparent, meaning they are not visible to users. They should function like plug-and-play devices;
you insert them into your network, and everything still functions the way they functioned before. Transparent bridges
can be created using two algorithms:
1. Backwards Learning.
    - Stop traffic from being sent where they are not needed.
2. A Spanning Tree Algorithm.
    - Break loops that may be formed when switched are cabled together randomly. 

#### Backwards Learning
Backwards learning ensures that data is only sent to where it's needed. When one of the devices connected to a switch
sends some data, it will be broadcast to all other connected devices. At this point to switch will only know who the
sender was. Now when another device wants to send some data to the device that just sent something, the switch will
already know where that device is. The switch pretty much remembers which device is where after these devices send a
message. This makes it so that whenever a device is the recipient of a message, it can be sent just to them, instead of
being broadcast on the network.

The disadvantage of this configuration is that you cannot change ports - the switch will remember that port A belongs
to device X, so connecting device X to port B would mean it would no longer receive any of the messages it is addressed
to as they will still be sent to only port A.

Another issue is that loops in the network would cause an infinite loop of broadcasts.

#### Spanning Tree Protocol
The spanning tree protocol (**STP**) ensures that no infinite loops of broadcasts occur. It does this by having every
port be in either a *forwarding* state, or a *blocking* state. Blocked ports will not send or receive any data, meaning
that properly blocking some ports will prevent broadcast loops.

The steps of the spanning tree protocol are as follows:
1. Select a *Root Bridge*.
    - This is done with a BPDU (bridge protocol data unit) packet containing a *Bridge ID*: bridge MAC Address + a
    Priority Number
        - The bridge with the lowest bridge ID becomes the root bridge.
2. Set all outgoing root bridge ports to forwarding mode (these are *designated* ports/interfaces).
3. Each non-root bridge sets a root port, and sets them on forwarding mode.
    - The port with the shortest path to the root port (using Dijkstra's algo. Each cable has a certain cost depending
    on their bandwidth).
4. Block port between two non-root bridges.
    - The port of the bridge with the highest bridge ID will be blocked (*non-designated*).

Now your bridges are loop free!
