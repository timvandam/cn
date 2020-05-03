# MAC Sublayer
The MAC (*Medium Access Control*) sublayer is responsible for deciding who can use the communication channel, it resides
at the very bottom of the Data Link Layer.

## Index
- [The Channel Allocation Problem](#the-channel-allocation-problem)

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
