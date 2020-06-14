# Security
Security deals with protecting a computer system from intentional misbehavior. This differs from random faults and
errors. While error codes can help you detect the latter, the former can still happen.

## Index
- [Ethical Hacking](#ethical-hacking)
- [Terminology](#terminology)
- [Attacker Model](#attacker-model)
- [Cryptography](#cryptography)

## Ethical Hacking
When identifying security problems in software, it's important to research these vulnerabilities legally and ethically.
You should not hack a bank to test its security without permission, etc., as circumventing protection mechanisms of a
network or computer is illegal in that case (in most countries in the world). If you download a copy of some software
you *are* legally allowed to test that software for vulnerabilities. However, exposing vulnerabilities to the public is
a grey area. When you find a vulnerability, there are three courses of action you could take:
1. Non-disclosure;
    - You don't disclose the vulnerability you found. This generally has no legal consequences, unless you were
    responsible for this this, and non-disclosure had far-reaching consequences (someone died, etc.).
    - Ethically this is questionable. It leaves users vulnerable to people with a malicious intent.
2. Full disclosure;
    - You publicly announce the vulnerability without first consulting with the developers. This is legal given you find
    this on your own device, as mentioned earlier.
    - Ethically this can be questionable, as you let both the developers and malicious parties know of a vulnerability
    at the same time.
3. Responsible/Coordinated Disclosure.
    - You first inform the developers. If they don't fix the issue in a timely manner you inform the public. This is
    legal as before. This gives the developers time to inform **Computer Emergency Response Teams** (CERTs), that have
    experience in setting deadlines and dealing with developers.
    - This seems more ethical than full disclosure, as you give the developers time to fix a vulnerability before
    malicious parties can take advantage of it.

## Terminology
Achieving security (protection against malicious use) entails achieving the following three properties:
1. Confidentiality: Ensure that only authorized parties can access information;
2. Integrity: Ensure that users can verify that they have the correct information;
3. Availability: Ensure that information and services are available when needed.

These three goals are often referred to as the CIA Triad. Note that availability indicates that whatever you want to
access should still be accessible when malicious parties are overflowing the entry point (e.g. DDoS attacks). This is
often impossible, though.

In addition to these three, there are a number of less prominent goals and properties that relate to security. Think of
*authentication* and *privacy*.

## Attacker Model
An *attacker/adversary model* describes the goals and capabilities of malicious parties. Generally a system protects
against all malicious parties following such a model. Hence, there is need for many of such models to ensure security.

The following terminology is typically used to characterize adversaries:

### Adversary Positioning
- **Internal** - an internal adversary controls participants in a system. An example is a corrupt server;
- **External** - an external adversary observes the system from the outside. An example is a malicious party listening
in on wireless data transfer;
- **Global** - an adversary that can observer or control the complete system. Examples are ISPs, as they can control an
entire autonomous system;
- **Local** - an adversary that can observe or control only a part of the system. An example could be Brian from IT
(internal local).

### Adversary Behavior
- **Passive** - the adversary does not manipulate protocol. E.g. a keylogger. A malicious service provider that follows
protocol but tries to gain confidential information is called a *honest-but-curious* adversary;
- **Active** - the adversary manipulates protocol. E.g. changes data;
- **Static** - the adversary doesn't change its behavior over time;
- **Adaptive** - the adversary changes its behavior over time.

### Computational Power of the Adversary
- **Computationally Unbounded** - the adversary has unlimited computational power. In practice this is unrealistic, but
can be useful when considering worst-case scenarios;
- **Polynomially Bounded** - the adversary can only compute algorithms that have polynomial complexity. If you use
sufficiently large inputs, eventually the adversary will no longer be able to handle it (unless its polynomial). E.g.
the adversary can execute O(n<sup>2</sup>) algorithms but not O(2^<sup>n</sup>) algorithms. 

Cryptographic algorithms that protect against computationally unbounded provide *information-theoretic* security while
algorithms protecting against polynomially bounded adversaries provide *computational security*.

### Dolev-Yao
The most common adversary model is the Dolev-Yao model, which defines a polynomially bounded active adversary, who can
overhead, discard, create, modify, delay and replay messages.

## Cryptography
We will need a few definitions to talk about cryptography:
- **Cryptosystem** - a set of algorithms that guarantee security properties;
- **(Cryptographic) key** - a parameter describing which algorithm to use;
- **Cryptography** - the research area of creating cryptosystems;
- **Cryptanalysis** - the research area of breaking cryptosystems;
- **Cryptology** - cryptanalysis + cryptography combined;
- **Cipher** - cryptosystem for encryption;
- **Plaintext** - original message `M`;
- **Ciphertext** - M after encryption;
- **Kerckhoff's Principle** - security should not depend on secrecy of algorithm, only on secrecy of keys.

### Symmetric-key Encryption
Symmetric-key encryption is encryption where a key does both the encrypting and the decrypting. Examples are
substitution ciphers, where every on every character in a text you apply an offset/key `K`. Transposition ciphers are
also symmetric-key encryption algorithms, but they simply rearrange characters instead of changing them. These two
examples are not very secure.

There are two types of symmetric-key encryption:
1. Stream Ciphers - apply encryption to each individual bit;
2. Block Ciphers - apply encryption on the input per block (of arbitrary length).

#### One-Time Pad
One-Time Pad is a secure encryption algorithm that even works against computationally unbounded adversaries.
Unfortunately, it does not work against adversaries that can read minds.

It works like this:
1. Create a random bit string as key (called the one-time pad);
2. Convert a message to a bit string;
3. XOR both strings to obtain your ciphertext.

The disadvantage to this approach is that the length of your ciphertext is the same as that of your original message.
This also means that your key has to be the same length as your message, which is not great. Additionally, it is much
harder to generate a random bit string than you might think. It is also possible to slowly gain more information on
your key as you keep using it. For example, if you're confident that the first word in a message was `hello`, you
already know quite a bit about the key. As you keep intercepting ciphertext you will be able to reconstruct more of the
key. This is why the random bit string should be used just once.

#### Pseudo-Random Number Generator (PRNG)
PRNGs generate deterministic 'random' numbers based on an input seed. PRNG can provide you with a stream of random bits,
so they are ideal for generating a one-time pad, for instance. You can now use PRNGs to use the one-time pad algorithm
without having to memorize and share the key for every message you want to send. Instead, you just share the seed for a
PRNG that generates you one-time pad.

To get rid of the deterministic nature of PRNGs you could use a *nonce*, a random number you XOR with your seed/key
before using it in your PRNG. Your nonce can be public, as long as the other party knows it. TODO: Figure out how the
nonce makes a PRNG nondeterministic, question asked on BS.

#### Blockciphers
The previously discussed algorithms were all bitwise, but this is not the only way of applying encryption. Blockciphers
use two algorithms: one for encrypting blocks and one for dividing and combining blocks (cipher modes).

##### Cipher Modes
There are multiple ways of dividing and combining data into blocks:

##### Electronic Code Book
This simply involves encrypting each block individually, creating block ciphertexts. Once you have computed all block
ciphertexts you can simply concatenate them to get your final ciphertext.

The reason why this cipher mode is not great is that this mode reveals patterns; two of the same blocks will yield the
exact same ciphertext.

##### Cipher Block Chaining
Cipher block chaining works by selecting an *initial vector* (IV). This IV acts as a nonce in that it adds some
more randomness to your encryption. Your message will now first be XORd with your IV before encrypting it. Using the
same IV for every block encryption would not help at all, so every computed ciphertext is used as the IV for the next
block encryption.

The problem with this approach is that this type of encryption cannot be parallelized.

##### Counter
If you *do* want to parallelize encryption and decryption you could use the counter mode. In this mode you still choose
an IV, but of half your block-length instead. You now encrypt your IV with your counter, and XOR the result of that with
your message to get your block ciphertext. The resulting block ciphertext will *not* become the next IV, as this would
once again prevent parallelism.

There are more modes, but these are the three most common ones.

---

How blocks are actually encrypted would take a long while to explain, so it is skipped in this course. It might be
explained in the Cryptography elective in year 3 (?). The following components make it happen, though:

##### P-Box
Permutation box. Takes an input and returns it as a scrambled output. Pretty much a physical transposition cipher.

##### S-Box
Substitution box. Takes an input and returns a substituted version of that input. E.g. 1001 becomes 1010. Every sequence
will be mapped to another, so in case of 4-bit sequences there would be 2<sup>4</sup> = 16 mappings. 

Block ciphers are a combination of these components.

#### DES
Data Encryption Standard. Takes a 64-bit input and output. Effective key length is 56 bits. DES used to be standard
from 1977 till the late nineties. Sometimes DES is still applied in the form of triple-DES, where you apply DES three
times, using a different key every iteration. However, the second iteration uses a DES decrypt instead of encrypt.

#### AES
Advanced Encryption Standard. Improvement on DES created in the early 2000s. AES features:
- Complete public design and algorithms;
- Key length of 128/192/256;
- Both hardware and software implementation possibilities.

### Asymmetric-key Encryption 
Asymmetric-key encryption is very similar to symmetric-key encryption, however, instead of having just one key for both
encryption and decryption, you now have separate keys for encryption and decryption. Generally the key used for
encryption is the public key, while the key for decryption is the private key. This means that you can share your
public key to anyone that wants to send encrypted messages to you, and only you will be able to decrypt it (with your
private key).

Asymmetric key encryption can't rely on XORs like symmetric key encryption does, instead something called *Euler's
function* is used.

#### Euler's Function
You have two primes, `p` and `q`, and `n = p * q`.

Euler's function states:

*φ(n) = (p - 1)(q - 1)*

For some math reason, the following equality holds for Euler's function:

*m<sup>e</sup> mod n = m<sup>e mod φ(n)</sup> mod n*

From this follows that:

*e \* d mod φ(n) = 1 -> m<sup>e \* d</sup> mod n = m*

#### RSA
Euler's function is used for RSA key generation. It works like this:
1. Choose two large (e.g. 1024 bits) primes `p` and `q`;
2. Compute `n` and `φ(n)`;
3. Choose a number `d`, relatively prime to `φ(n)` (so *gcd(d, φ(n)) = 1*);
4. Find `e` such that `e * d = 1 mod φ(n)`.

Your public key will now consist of (e, n), and your private key will consist of (d, p, q).

RSA Encryption will now work as follows:

Encrypting `M` yields ciphertext `C` = *M<sup>e</sup> mod n*.

Decrypting `C` yield `M` = *C<sup>d</sup> mod n*.

As you can see, encrypting requires (e, n), and decrypting requires (d, n) = (d, p, q).

Luckily, you don't have to know all this math stuff.

RSA is computationally secure if:
1. Factorization of `n` is computationally infeasible;
2. Deriving the `e`-th root modulo `n` is computationally infeasible. 

RSA less efficient than symmetric key encryption, as it requires exponentiation instead of simple XORs. Keys also have
to be much longer for it to be secure (2048 bits vs 256 bits). Since slow encryption is undesired in situations where
much data is constantly being generated and transferred, asymmetric encryption is usually only used to communicate the
key used for symmetric encryption, which will then be used. This is called *hybrid encryption*. An example of this is
Diffie-Hellman.

### Integrity and Authentication
Encryption alone hides messages from eavesdroppers, but does not guarantee that what was sent was not altered. Luckily
cryptography offers yet another solution: integrity. **(Data) Integrity** verifies that the received data is correct.
**Authentication** makes it possible to verify identity.  

A few more terms:
- **Anonymity**: the ability to hide your identity;
- **Nonrepudiation**: the inability to refute the validity of a statement;
- **Repudiation**: the ability to plausibly deny a statement.

There are a few methods to ensure integrity:
- **Message Authentication Codes** - uses symmetric cryptography;
    - Data Integrity;
    - Authentication;
    - Repudiation.
- **Digital Signatures** - uses asymmetric cryptography.
    - Data Integrity;
    - Authentication;
    - Nonrepudiation.

#### MAC
MAC means that you send an additional code that only the sender can generate (with a secret key), so any malicious third
parties won't be able to send a message with a valid MAC unless they somehow know the key. MACs are usually generated
using hash functions. Usually the message and key are somehow combined to generate a MAC.

Good hash functions have a few requirements:
- **Collision Resistance**;
    - Polynomially bounded adversaries should not be able to find collisions.
- **Preimage Resistance**;
    - It is hard to reverse a hash (i.e. go from a hash to the original value).
- **Second Preimage Resistance**.
    - It is hard to find a value that yields the same hash as another value.

#### Digital Signatures
The problem with MAC is that you can only talk to one person at a time. If you want to speak in a group of three people,
each person would have to know the key with which the MAC is generated, making it possible for either one of these three
people to send a message claiming to be another person. Digital signatures solve this issue by having different keys for
each party. Everybody now has their own public and private keys. Signatures are then encrypted using a private key,
allowing third parties to decrypt the signature using a public key to verify the signature. RSA can be used to create
signatures, but there are other methods as well.

The disadvantage pf digital signatures is that it's slow due to its asymmetric nature. The solution to this is something
called *Hash-then-Sign*, where you simply hash whatever you want to sign, and then sign that hash. You now only have to
sign a hash instead of a big message. 

---

Given the previous integrity methods, it is still possible to simply change a key on a web page to fake a signature. One
way of solving is this is by involving a third party: a certification authority (CA).

#### Certification Authority (CA)
CA's are parties everybody trusts, such as a government or central organisation. The CA signs public keys, pretty much
meaning that this CA vouches for this public key. People can then validate signed public keys using the CA's public key
to ensure the key you're dealing with is valid. A protocol that uses this is *X.509*.

In the real world there are several *root CAs*, which can give certificates to first-layer CAs, which can give
certificates to parties lower in the chain, eventually ending up at websites. This hierarchical structure kind of
resembles the way DNS is structured. This infrastructure is called the PKI or *Public Key Infrastructure*.

To verify a the signature of a website's certificate, you can now verify the signature of the certificate of the site's
certificate's issues, then their certificate's issues, and so on until you get to the root authority. The path from a 
certificate to the root authority is called the *certificate chain* The root authority (or authorities) are usually
embedded in your browser or computer.

##### X.509
X.509 has a few requirements:
- Certificates should be easy to read;
    - Allows for manual verification;
    - Makes it easy to compute digitally = fast.  

X.509 specifies an encoding method and a number of fields:
1. Version;
2. Serial number;
3. Signature algorithm;
4. Issues;
5. ...

### Secure Wireless Communication
Wireless communication is particularly vulnerable to attacks, as all communication happens through the air making data
easy to intercept.

Secure wireless communication requires a few key properties:
- Data integrity;
- Confidentiality of content;
- Access control.

The first attempt at doing this was WEP (Wired Equivalent Privacy). It did not live up to its name.

#### WEP
WEP attempted to ensure data integrity with CRC. The problem is that you can simply change the data and compute the CRC
for that data.

Confidentiality was "ensured" using a 40-bit key stream cipher. This is a problem because 40-bit keys are essentially
brute-force-able. They eventually changed the key length to 104 bits, but the random number generator WEP used was
broken so this did not help.

Access control was done using passwords. Great, expect that every user has the same password. This is fine if you're
trying to protect your home Wi-Fi, but in big cooperation it would take just one person sharing the password to
circumvent access control. It was also very easy to listen in on devices authenticating themselves to the Wi-Fi. WEP
would send you a challenge `c`, which you had to XOR with a keystream and send back. You can simply listen in on both
messages and XOR them to get the keystream! This isn't exactly the password, but rather a derivative of the password.
Yet you can still use this to authenticate yourself and get access to the network. The keystream is a stream of
pseudorandom numbers with a seed of key XOR nonce. This nonce was also sent in the authentication process, making it
re-usable by Malory.

#### WPA
When people figured out WEP was terrible, they switched to WPA: Wi-Fi Protected Access. We're currently at WPA2 (since
2006). It is possible to perform an attack on WPA2s handshake, though, called the Krack attack (2017). The reason this
attack become possible in the first place was that the handshake was vulnerable to replay attacks. To solve this an
attempt was made to create WPA3, but it was quickly cracked by the same guy that designed the Krack attack.

### Virtual Private Networks (VPNs)
The following section is sponsored by NordVPN.

If you want to safely communicate over a network of which you don't know whether it's safe, you could use a VPN to
ensure that this network cannot read your traffic. This works by utilizing IPsec (IP Security). IPsec uses multiple
protocols, but only ESP will be discussed.

#### Encapsulating Security Payload (ESP)
ESP provides *confidentiality*, *integrity*, and *replay attack protection*. It does so by taking a normal IP packet and
modifying it. This can be done in two modes: **transport mode** and **tunnel mode**.

##### Transport Mode
Transport mode works by encrypting the body of IP packets and adding a (H)MAC for authentication. The problem with this
mode is that the IP header can still be modified. The result of this is that the person receiving your message *must* be
able to decrypt the payload in order to use it. So transport mode might be great for private companies that want to
ensure secure traffic to and from their servers, but will not be great for general internet users, as for that to work
every server would have to use the same method of encrypting data, etc.
 
##### Tunnel Mode
Tunnel mode works by prepending a new IP header before the actual IP header. The old IP header will then be part of the
(encrypted) payload. The new IP header will cause the packet to be sent to a different server, which will decrypt your
payload and send the packet inside that payload. Its response is encrypted once more before sending it back to you. In
this mode you are practically using proxies to send your traffic for you, so only that proxy can see your traffic
instead of the network you are sending over. This proxy is called the **VPN Gateway**. When you enable a VPN your device
will also function as a VPN Gateway. In transport mode the server you are sending to must be a VPN gateway.

### TLS and DNSSEC
Transport Layer Security ensures confidentiality and integrity on top of (usually) TCP. It is mostly used on the web by
HTTPS. TLS is the successor of SSL.

Tls connections are established using *ciphersuites*. Ciphersuites are essentially a combination of symmetric-key
encryption and MAC. If you want to establish a TLS connection you first send you TLS version and all ciphersuites you
know/understand. The server will then choose a version and ciphersuite (the most secure one that both you and the server
understand). Additionally, the server will send its X.509 certificate chain which you will verify. You then perform a
key exchange using a specific key exchange algorithm, or Diffie-Hellman. TLS gives you the following security
properties:
- Server authentication;
- Message integrity;
- Message confidentiality;
- Optionally client authentication.

#### DNS Spoofing
DNS uses a solution similar to TLS to prevent spoofing (= tricking a nameserver to provide an incorrect IP address).
When a DNS cache contains a malicious entry this cache is called a *poisoned cache*. The solution is DNSSEC, which uses
a hierarchical PKI and digital signatures, but no encryption. So it's pretty much TLS but with digital signatures
instead of MAC.

DNSSEC has some problems with backwards-compatibility, so it is not being widely used at the moment.

### Pretty Good Privacy (PGP)
PGP is a program for email security that uses hybrid encryption to ensure confidentiality. integrity and authentication
using digital signatures. In short, PGP does hybrid encryption followed by hash-then-sign.

In PGP, each user maintains two key pairs. One private key for signing, with a public key for verification, and a public
encryption key with a private key for decrypting. The keys used for the asymmetric encryption are used to encrypt
another key that is used for symmetric-key encryption, which is applied on the actual message + signature.

### Multi-factor Authentication
TLS authenticates web servers to users, but users should also be able to identify themselves. Passwords can be pretty
safe if you have a good password, but they are not nearly as safe when not used in combination with multi-factor
authentication (MFA). MFA involves performing a few more checks before believing that the user that entered the correct
password is who you think it is. This creates for strong client authentication that should generally be used by any
risky service such as online banking. Factors can be:
1. Something you know - passwords, pins, passphrases, etc.;
2. Something you have - your phone number, credit card, physical key, etc.;
3. Something you are - fingerprints, iris scan, face scan, etc.
Typically, you should use more than just one factor (generally a password/pin). Sometimes factors such as location can
also be used, but this should not be used to authenticate a user, but rather to detect irregularities which you could
use to trigger an even more comprehensive authentication scheme.
