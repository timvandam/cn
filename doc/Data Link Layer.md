# Data Link Layer
The data link layer is responsible for regulating data flow, detecting or correcting errors during transmission
(via the physical layer). It groups bits into so-called **frames**, and provides services to the layer above, the
network layer.

## Index
- [Error Detection](#error-detection)
- [Services](#services)
- [Framing Methods](#framing-methods)
- [Acknowledgements and Repeats](#acknowledgements-and-repeats)

## Error Detection
The physical layer is not perfect. Many things can go wrong; bits can be flipped, omitted, etc.
Networks today usually use three different error detection algorithms: **checksums**, **cyclic redundancy checks** (CRC)
and **message authentication code** (MAC). Error detection algorithms can either only detect errors, or also locate them.
Being able to locate them is very useful for error-prone mediums like WiFi, as a complete retransmission could very likely
also contain errors.

### Checksum
A checksum is simply a sum of all the values of length `n`  in the packet. Checksums are very cheap to compute, however they are not
very robust because two (or more) errors can cancel each other out. This makes it only viable for catching one-bit errors
TCP & IP use checksums.

`0x10100428`'s checksum where `n = 16` would result in a checksum of `0x1010` + `0x0428`. Depending on how numbers are
represented the result will vary (IP, UDP, TCP use one's complement)

### Cyclic redundancy code
A CRC is much more expensive to compute, however it is also much more robust than a checksum.
It computes the remainder of a polynomial. If your CRC is `c` bits in length, it will be able to detect any burst
less than or equal to `c` bits in length.
It is always able to detect an odd number of errors, or 2 errors.
There is a 2<sup>-c</sup> chance one packet's CRC matches another.
Link layers typically use CRC's, which is why TCP & IP can get away with checksums. USB and Bluetooth use 16-bit CRC's.

CRC's use polynomial long division. The polynomial used is called the *generator polynomial* `G`. It should have degree `c`.
`G` is always left-padded with a 1. The strength of a CRC algorithm depends on `G`.

To generate a CRC, you take the message `M`, right-pad it with `c` 0-bits, and divide it by `G`. The remainder is the CRC.
To check a CRC, you take divide `M + CRC` by `G`. If the remainder is 0, then the CRC passes.

### Message authentication code
MAC combines a packet `M` a some secret key `s` to generate a value. This keeps the data in the packet secure as only
the sender and receiver will know secret `s`. This prevents packets from being forged/modified by a third party.
Third parties *can* however replay the packet when captured. MAC is not great for detecting errors; when using a strong
MAC, if you flip a single bit there is a 2<sup>-c</sup> chance the packet will have the same MAC.
A 16-bit CRC will detect any burst of errors that is 16 bits or shorter. A strong 16-bit MAC has a 1/65536 (0.02%) chance
of not catching errors. This seems low, however the amount of packets sent just watching a video is so high that the
0.02% failure rate will result is a lot of undetected errors.
TLS uses MAC.

`c = MAC(M, s)` and `|c| << |M|` where `c` is the code.

### Which algorithm detects which errors
| Algorithm | 1 bit error | 2-bit burst error | 9-bit burst error | 2 bit errors (100 bits apart) |
|---|:---:|:---:|:---:|:---:|
| 8-bit checksum | Y | N | N | N |
| 16-bit checksum | Y | N | N | N |
| 8-bit CRC | Y | Y | N | N |
| 16-bit CRC | Y | Y | Y | N |
| 32-bit MAC | N | N | N | N

That doesn't seem promising... So why bother detecting errors anyway?

In practice you use multiple layers of errors detecting:
- The link layer uses CRC
- IP uses checksums
- TCP uses checksums
- Applications can have their own error detection

This minimizes the chance of errors by a whole bunch, making error detection worth it.

### Parity Check
The parity check can be used to detect and correct 1-bit errors (when using parity blocks). It is a very simple example
of a 1-bit CDC, using the generator polynomial `x + 1` (`n = 1`). The parity check can be done in two ways; even
parity checking or odd parity checking.

When applying even parity checking, the 1-bit CDC (the parity bit) will be used to ensure that there are an even amount
of 1's in the packet. This means that `1001` would have the parity bit `0`, and that `1101` would have the parity bit `1`.

Odd parity checking works in the exact same fashion, but this time ensures that there is an odd amount of 1's in the packet.

When two bits flip (or any even amount of bits), the parity check won't work anymore.
So it can only be used to detect 1-bit errors.

#### Parity Blocks
Parity blocks (2-dimensional parity checks) are used to detect where a 1-bit error occurred within a packet. Instead of
sending just one parity bit per packet, an additional one is sent for each bit sent per packet. These are sent as a last
packet.

This could look like this:

|Data|Parity Bit|
|:---:|:---:|
|11001010|0|
|10010110|0|
|10011011|1|
|||
|11000111|1|

The last packet that was sent will now contain additional parity bits for each previous packet. This makes it possible
to see exactly where the error occurred (if an error occurred).

In a case where 3 bits flip in a single packet, it would still be possible to detect their positions when using parity blocks.
However, when multiple packets in the block have flipped bits, this is no longer possible. This once again means that
it is only possible to detect and correct 1-bit errors.

### Hamming code
Hamming codes can be used to correct single-bit errors. They work by using multiple parity bits, each at the 2<sup>n</sup>-th
position of a packet. These parity bits account for every alternating block of n bits after the parity bit (including the parity bit itself)
(first n bits count, next n don't, next n do, etc.). When receiving a packet using a hamming code, the sum of the positions
of incorrect parity bits will be the position of the incorrect bit. This looks like this:

|p1|p2|m3|p4|m5|m6|m7|p8|m9|m10|m11|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|0|0|1|1|0|1|1|0|0|0|0|
|&check;|&cross;| |&cross;| |&cross;| |&check;||||
||||||||||||
|1|1|1|1|1|1|1|1|1|1|1|
|&check;|&check;| |&check;| | | |&check;||||

#### Burst errors
Hamming codes can also be used to detect bursts of errors. This is done just like parity blocks. You send packets per column
of bits instead of per row. After having received the whole matrix of packets it is then reconstructed and the parity bits
are checked. This works because columns are sent instead of rows. When a burst error occurs in this column there will be
just one mistake per row, meaning that rows can be fixed again. If there are burst errors in multiple columns, however,
this won't work.

## Services
Layers can offer two types of services to layers above them; connectionless services and connection-oriented services. 

### Connectionless Services
Connectionless services are services that perform tasks without need for communication. An example is a postal service.
You mail something to somebody, and it will eventually arrive. Other messages to that person might arrive sooner, later,
you don't know. Every frame is independent of all others, so it is possible to send two frames where the second to be
sent can arrive first. Since there is no negotiation connectionless services are more prone to data loss, however this
also increases speed. An example could be UDP. Connectionless services are generally called **datagram services**.

### Connection-oriented Services
Connection-oriented services function sort of like a phone; to talk to somebody you pick up the phone, dial their number,
talk, and then hang up. Similarly, connection-oriented network services first establish a connection, then negotiate
parameters (such as the maximum frame size), then communicate and eventually hang up. Negotiations make it possible to
ensure that all frames arrive in order, and that they all arrive at all. This makes a connection-oriented service
important when downloading files, for instance, as you can't do anything with a half-downloaded game, zip, etc. This
overhead makes connection-oriented services less speedy. An example could be TCP. Connection-oriented services are
generally called **telegram services**.

Reliable connection-oriented services has two minor variations: message sequences and byte streams.

### Message Sequences
When using message sequences the length of each message is preserved. They will never be combined to create one big
message consisting of both messages. E.g. when sending two 1024-byte messages they will also arrive as two 1024-byte
messages, not as one 2048-byte message.

### Byte streams
Byte streams are exactly what their name implies. Data doesn't have clear boundaries, you don't know when a message
starts and when a message ends. The two messages would then arrive as one 2048-byte message, and you won't know how
exactly those bytes were sent: as two 1024-byte messages, one 2048-byte message, or even 2048 1-byte messages.

Both byte streams and message sequences have their advantages and disadvantages, so which is better depends on the
context they're used in. When using VoIP, for instance, you want to hear what the other party has to say as soon as
possible. In that case the overhead connection-oriented services have should be prevented as it slows down the
communication. A little bit of jittering is a valid tradeoff of connectionless services in that case (instead of
buffering). The same goes for transmitting live video.

## Framing Methods
The datalink layer takes data from the network layer and divides it into frames. To indicate the start and end of a
frame, the sender has to send some additional information. An algorithm that does this is called a *framing method*.

### Prepending Length
It might seem easy to indicate the start and end of a frame; simply prepend the length of the frame (+ the length
value's length). However, this is not effective as you might think, since one lost bit, or flipped bit in the frame length
will cause everything sent after to be misinterpreted. 

### Byte stuffing
Another framing method is indicating the start and/or end of a frame using a byte-size flag, which is simply a specific
sequence of bits. Like the *byte count* method just discussed, this problem also suffers from bit flips and missing bits.
However, unlike the byte count method this method won't misinterpret every message sent after an invalid one; instead it
will read too much until it encounters another start/stop sequence. This makes it a bit better than simply prepending the
amount of bytes, but is still not an ideal solution. Another disadvantage is that the start/stop sequence will no longer
be usable in the actual message.

This can kind of be fixed by prepending an escape sequence before the data sequence that is also the start/stop sequence.
This escape sequence, however, can also suffer from bit flips. Additionally, what happens if your frame contains the
escape sequence? Then you would have to escape the escape. If your frame contains the escape + start/stop, then you
would have to escape twice. Simply put, it all becomes a mess of escapes! You're effectively trading frame length for
reliable reading.

### Bit stuffing
Byte stuffing can be very space inefficient, as any reserved sequence in your message will take up twice as much space
as you have to escape those. With bit stuffing this is not the case. Instead of a flag byte, there will be a flag
sequence of indeterminate length, and instead of an escape byte you use an escape bit. A great advantage of bit stuffing
is that it can be implemented at the hardware level. To escape a flag you simply insert a 0-bit into the sequence. This
new sequence - that starts the same way the flag sequence does but contains an extra 0 at some position - can be
recognized by the receiver, who will remove the redundant bit. Not only the sequence gets an additional 0-bit, but every
sequence starting with a certain combination of bits.

## Acknowledgements and Repeats
If your aim is to have reliable communication, acknowledgements and repeats are an important concept. If your device is
constantly listening for messages, it is important to let the sender know when you've received the message to ensure no
messages are lost. This is done by sending an acknowledgement (ACK). If the sender doesn't receive an ACK, then it will
simply resend the lost message. This is called **Automatic Repeat ReQuest (ARQ)**. To ensure that it is known which
messages were acknowledges and which weren't, messages are numbered using *sequence numbers*. This also allows all
message to be processed in the order they were sent. When applying **stop and wait** only one outgoing message can be
active at a time. In this case you need just 1 bit to track a message - one for a new message, and one for a
retransmission.

Instead of sending an acknowledgement for every received message, you can also combine an acknowledgement with an
outgoing message to improve performance. This is called **piggybacking**. You should not wait too long before sending the
acknowledgement on its own, though, because then the sender would retransmit the received message as it didn't receive
an ACK.

### Sliding Window Protocols
Sliding window protocols limit the number of unacknowledged frames and introduce the use of buffers at the sender side.
An important aspect of sliding window protocols is their use of sequence numbers as discussed in the previous paragraph.
Sliding window protocols keep track of the status of frames in a buffer, having at most `k` transmitted but
unacknowledged frames at a time. The buffer starts with transmitted, acknowledges frames, then come transmitted
unacknowledged frames, and then come not yet transmitted frames.

The section of the buffer that contains unacknowledges but transmitted frames is called the *sliding window*. Whenever
the top frame is acknowledges, the window will slide down one, transmitting the first frame in the section of not yet
transmitted frames.

#### Choosing `k`
The choice of the value of `k` is important; it determines how many unacknowledged, transmitted frames your buffer will
contain. This means that `k` must not exceed the size of the buffer. However, how large `k` is can also have an effect
on the speed of applications. When there is a lot of latency, you will have to wait for acknowledgements. If the section
of the buffer containing unacknowledges, transmitted frames is full during that time your application will idle for a
bit. This is bad as it means your data transfer rate will be slower. So `k` should be set so that there are no periods
at which no bandwidth is used.

#### How about errors?
Until now any errors would break the sliding window, as frames that never get acknowledges would cause the sliding
window to be stuck. One solution to this is **Go-Back-N**.

Go-Back-N starts a timer the moment a frame is transmitted. Once the timer goes off and the frame has not received an
ACK, it will simply retransmit the unacknowledged frame and all frames sent after. This effectively means that the
sliding window is moved back up until the unacknowledges frame is at the top, and that all frames in the window are then
transmitted again.

An alternative strategy is **Selective Repeat**, where not all frames after the unacknowledged frame are sent, but only
the unacknowledged one is.  

Selective repeat introduces *Negative Acknowledgements*. Whenever the sender receives such a message it will resent 
frame it received such an NACK for. If a frame doesn't arrive, the receiver won't be able to send a NACK, though. In
those cases the receiver will detect a missing frame when receiving the next one. For example, when frame 3 goes missing,
and the receiver just received frame 4, it will send a NACK 3 to the sender. The receiver *will* buffer frame 4 though,
so contrary to Go-Back-N now just the lost frames will have to be resent.

Selective repeat can also use timers, just like Go-Back-N, but instead for individual frames. A combinating of NACK's and
timers is also possible.
