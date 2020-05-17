# Network Layer
The network layer forwards your messages from one end of the world to the other. It figures out how to get from A to B,
prevents network congestion (traffic jams), and provides *Quality of Service*. It is even possible to connect multiple
networks using the network we know as the Internet. This is all done using IP addresses (owned by ISPs).

## Index
- [Connectionless vs Connection-oriented](#connectionless-vs-connection-oriented)
- [Routing](#routing)
- [Congestion Control](#congestion-control)
- [Quality of Service](#quality-of-service)
- [Internetworking](#internetworking)

## Connectionless vs Connection-oriented
In the network layer there are two ways of sending packets. You either decide where to send each packet individually, or
use a predefined route. The former is called connectionless and the latter connection-oriented. The internet is
connectionless, as connection-oriented is not very scalable. One problem with connectionless is that it is difficult to
control congestion.

## Routing
When it comes to routing there are a few important (often conflicting) requirements:
1. Correctness - it should always function correctly;
2. Simplicity;
3. Robustness - it should quickly adapt to changes or disruptions;
4. Stability;
5. Fairness;
6. Efficiency.

### The Optimality Principle
When the best route from A to C goes through B, then the best route from B to C will follow the same path. In other
words: any subset of an optimal path between two points, will also be the optimal path for the (new) start/end points.
Because of this principle, the collection of all best paths to a given destination forms a tree.

There are multiple ways to route packets, e.g. *Distance Vector Routing*, *Link State Routing*, and
*Hierarchical Routing*. An important aspect to these algorithms are **Routing Tables**.

### Routing Tables
Routing Tables are tables that map a destination to a distance and point which can be used to get there. That point will
either be the destination, or will know where to forward your message to such that it will eventually reach its
destination.

An example routing table for a point C:

|To|Distance|Line|
|---|---|---|
|A|7|A|
|B|59|A
|c|0|-|

In this table A is directly connected to C, this can be derived from the fact that to get to A, you have to send it to A.
So no forwarding has to happen to get to A from C. It also shows that B is not directly connected to C. It is not known
whether B is directly connected to A, but it is known that it is possible to get to B through A.

### Distance Vector Routing
1. Send your *distance vector* to your neighbors (directly connected nodes);
    - A distance vector contains a node's distance to each node they know.
2. Use incoming distance vectors to construct a routing table;

This algorithm is nice and simple, but has a flaw: if you receive an invalid distance vector, or have outdated data, you
might get stuck in a loop, or traverse a path that doesn't end at the point you were sending to. This can make points
unreachable even though they're online.

Take this example: B can reach A with cost 1. It broadcasts this information in its distance vector to its neighbor D.
Suddenly A goes down. B notices and tries to find another way to reach A. Luckily it sees that D can reach A with
cost 2, great, lets update the cost to 3! Actually, not so great; D will receive a new distance vector and update its
cost to get to A to 4. B notices this and updates its distance vector again, D does it again, and this goes on forever.

This is called the *count to infinity* problem, as the cost to get to an offline node literally counts up infinitely.

### Link State Routing
Link state routing doesn't suffer from the count to infinity problem, but is more complicated. It uses a *shortest path
algorithm*.

1. Send information only about your direct neighbors;
2. Build an overview of the network and run a shortest path algorithm.

Great! NOT. When node A updates its information it will have to send that to every node in the network. When every node
in a network does this the entire network will flood in routing packets. This is absolutely not scalable.

It gets even worse when you receive old data after receiving new data. That completely throws off a routing table,
causing stuff to be routed via the wrong path which can either be slower, or have a dead end.

To prevent this you could use sequence numbers. NOT. You never know what the latest sequence number is as the network is
decentralized. This makes it prone to malicious users that send messages with very high sequence numbers, confusing the
entire network. A simple technique that prevents this is using a timeout number in addition to a sequence number.

### Hierarchical Routing
Hierarchical routing reduces the size of routing tables by subdividing a network into smaller networks. Each sub-network
is now seen as an individual machine.

## Congestion Control
When too many nodes try to communicate with each other the network gets congested. It's the responsibility of both the
network and transport layers to ensure that this doesn't happen.

### Increasing Bandwidth
There's a few ways of resolving congestion, the simplest of which is increasing bandwidth. This increases capacity, but
obviously doesn't prevent network congestion.

### Traffic Aware Routing
Traffic aware routing means choosing routes depending on traffic. This takes load off traffic-heavy cables and instead
tries to evenly balance the load. A downside is more latency.

### Admission Control
The name already reveals it: traffic needs admission, which only happens when there is room for more traffic. This
completely prevents congestion, but does at latency to the waiting traffic. This is effective just like traffic lights
before highways are.

Admission control can be combined with traffic aware routing.

### Traffic Throttling
Notify the sender when the network is congested in the hopes of their implementation waiting before sending more traffic.
This relies on the sender to be compliant! It's possible to do this with a so-called *choke signal*, which will be sent
as response to packets that routers that detected congestion will have marked. This is called **end-to-end traffic
throttling**. Since this relies on messages being sent downstream a congested network, it is quite slow.

An alternative is **link-by-link traffic throttling**. With link-by-link every router in the path of a marked packet
will start to throttle. This is an aggressive approach as even non-congested routers will slow things down. This also
causes buffering, consuming memory, and is still reliant on compliance.

### Load Shedding
Load shedding chooses partial failure over total system failure. It does literally what its name implies; it drops
packets whenever the network starts getting congested. It is very easy to implement, is reliable, but will cause packet
loss. Packet loss can now be an indication of congestion, which can be used by senders to slow down. When senders decide
to slow down the network congestion will go down again. This is very effective and fast as network congestion will
instantly go down as a result to packet loss.

#### Random Early Detection
Random Early Detection (RED) will drop packets if the buffer space is **almost** full. As just mentioned this is an
implicit signal for the sender to slow down.

## Quality of Service
The congestion techniques before are good when you need the best performance under the circumstances. Some applications,
however, require stronger performance guarantees (low latency, minimum throughput, etc.). This is where *QoS* comes in.

### Overprovisioning
The easing QoS solution is simply adding more capacity, this is called overprovisioning. The downside is that this
solution is very expensive. You would pretty much have to be able to meet the demand of all applications at the same
time to guarantee QoS for everyone (or actually just the amount of expected concurrent users).

---

Cheaper solutions that still provide QoS are called *QoS mechanisms*. To ensure QoS, four issues must be addressed:
1. What applications need from the network;
2. How to regulate traffic;
3. How to reserve resources at routers to guarantee performance;
4. Whether the network can safely accept more traffic.

Unfortunately there is no single technique that efficiently handles all these issues, however, some techniques at the
network and transport layer can be combined to cover them quite well.

A stream of packets from a source to a destination is called a **flow**. Each flow can be characterized by four
parameters: **bandwidth**, **delay**, **jitter** and **loss**, which together determine the QoS required by the flow.

Each of these parameters but *jitter* should be quite self-explanatory by now. Jitter is the variation of the delay.
Hence, jitter is an important parameter when data needs to arrive in consistent intervals. Examples would be live video
and audio; if every frame is delayed by 2 seconds its completely fine, you will just see everything two seconds after
the fact. If this delay changes, however, you will have to wait (unless the application does something smart).

Each application has different needs. Video conferences, for instance, have high bandwidth, delay, and jitter demands,
but loss is not that big of a deal as high quality video is not always that important. File sharing only has high
bandwidth requirements, as responsiveness doesn't really matter. Most video games *are* sensitive to delays, so their
delay requirements are higher while bandwidth is usually not that important as the packets they send are usually small.

## Internetworking
Sending packets over multiple networks poses a number of challenges:
1. Networks may use different protocols;
2. Networks may offer different QoS guarantees;
3. Networks have different maximum packet sizes.

### Tunneling
If the source and destination networks use the same protocol, we can use tunneling to send data through networks that
doesn't use this protocol. This requires the relaying networks to support both the protocol that the source &
destination use and the protocol that the networks in between the source and destinations use. A disadvantage of this is
that QoS guarantees are now no longer guarantees, as they depend on all routers in the path.

Another issue arises when the source wants to send a message with size 10, while the intermediate protocol only supports
messages of size 8. In this case *packet fragmentation occurs*, where packets are split into smaller parts. When
applying **transparent fragmentation** these are once again reconstructed before sending it to the final destination. A
disadvantage of this is that this is more expensive for routers to do, and adds complexity. Hence, there is also an
alternative in **nontransparent fragmentation**, where fragmented parts are delivered to the destination. The IP
protocol uses nontransparent fragmentation.

An alternative to packet fragmentation is **MTU Discovery**, where packets that are too large are simply not transmitted,
making the source responsible for splitting packets into smaller bits. This is also used in IP.
