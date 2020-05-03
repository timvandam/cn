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

