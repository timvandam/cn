# Data Link Layer
The data link layer is responsible for detecting or correcting errors during transmission (the physical layer)

## Index
- [Error Detection](#error-detection)

## Error Detection
The physical layer is not perfect. Many things can go wrong; bits can be flipped, omitted, etc.
Networks today usually use three different error detection algorithms: **checksums**, **cyclic redundancy checks** (CRC)
and **message authentication code** (MAC)

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
of a 1-bit CDC, using the generator polynomial `x + 1` (`n = 1`). The parity check can be done in two ways; the even
parity checking or odd parity checking.

When applying even parity checking, the 1-bit CDC will be used to ensure that there are an even amount of 1's in the
whole packet. This means that `1001` would have the parity bit `0`, and that `1101` would have the parity bit `1`.

Odd parity checking works in the exact same fashion, but this time ensures that there is an odd amount of 1's in the packet.

When two (or any even amount) bits flip, the parity check won't work anymore. So it can only be used to detect 1-bit errors.

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
