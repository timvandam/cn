### Which is not an OSI Layer?
The "Chocolate Layer". The data link layer handles data validity/error detection. The network layer provides means of
transporting packets, or routing it through the internet. The application layer is closest to the end user, and will do
whatever is needed for the application of the received/to be sent data.

### Which layer is malfunctioning when data isn't being forwarded (using IP routing)
The network layer; it's responsible for the network through which data is transferred.

### Why are heirarchical networks unsuitable for real-world networks?
They are vulnerable to attacks - the moment a system in the network is down, all systems will be split into parts that
can no longer communicate with each other.

### What does CRC offer?
CRC can detect 1-bit errors, any odd number of errors, and bursts of errors smaller than or equal to the CRC's length.
It can not be used to detect errors. So it can detect but not correct some errors.

EDIT: While this is true for standard CRC, extended CRC can correct errors by changing the message to the closest
multiple of the generator. So it could also be argued that it can detect and correct most errors.

### Using parity blocks, the check for the third row and the bottom right bit are wrong. What happened?
The 3<sup>rd</sup> row's parity bit was flipped, making it seem like that row is the 'crulpit'. This is shown by the
bottom right parity bit as it accounts for the last row and the last column. 

EDIT: It is also possible that a message bit was flipped in the 3<sup>rd</sup> row, and that its respective column
parity bit was also flipped. This would also lead to an incorrect bottom right parity bit and an incorrect
3<sup>rd</sup> row parity bit.
