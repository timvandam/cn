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
- **(Cryptographic) key** - a parameter which algorithm to use;
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

### Pseudo-Random Number Generator (PRNG)
PRNGs generate deterministic 'random' numbers based on an input seed. PRNG can provide you with a stream of random bits,
so they are ideal for generating a one-time pad, for instance. You can now use PRNGs to use the one-time pad algorithm
without having to memorize and share the key for every message you want to send. Instead, you just share the seed for a
PRNG that generates you one-time pad.

To get rid of the deterministic nature of PRNGs you could use a *nonce*, a random number you XOR with your seed/key
before using it in your PRNG. Your nonce can be public, as long as the other party knows it. TODO: Figure out how the
nonce makes a PRNG nondeterministic, question asked on BS.

### Blockciphers
The previously discussed algorithms were all bitwise, but this is not the only way of applying encryption. Blockciphers
use two algorithms: one for encrypting blocks and one for dividing and combining blocks (cipher modes).

#### Cipher Modes
There are multiple ways of dividing and combining data into blocks:

#### Electronic Code Book
This simply involves encrypting each block individually, creating a block ciphertext. Once you have computed all block
ciphertexts you can simply concatenate them to get your final ciphertext.

The reason why this cipher mode is not great is that this mode reveals patterns; two of the same blocks will yield the
exact same ciphertext.

#### Cipher Block Chaining
Cipher block chaining works by selecting an *initial vector* (IV). This IV acts as a nonce in that it adds some
more randomness to your encryption. Your message will now first be XORd with your IV before encrypting it. Using the
same IV for every block encryption would not help at all, so every computed ciphertext is used as the IV for the next
block encryption.

The problem with this approach is that this type of encryption cannot be parallelized.

#### Counter
If you *do* want to parallelize encryption and decryption you could use the counter mode. In this mode you still choose
an IV, but of half your block-length instead. You now encrypt your IV with your counter, and XOR the result of that with
your message to get your block ciphertext. The resulting block ciphertext will *not* become the next IV, as this would
once again prevent parallelism.

There are more modes, but these are the three most common ones.

---

How blocks are actually encrypted would take a long while to explain, so it is skipped in this course. It might be
explained in the Cryptography elective in year 3 (?). The following components make it happen, though:

#### P-Box
Permutation box. Takes an input and returns it as a scrambled output. Pretty much a physical transposition cipher.

#### S-Box
Substitution box. Takes an input and returns a substituted version of that input. E.g. 1001 becomes 1010. Every sequence
will be mapped to another, so in case of 4-bit sequences there would be 2<sup>4</sup> = 16 mappings. 

Block ciphers are a combination of these components.

### DES
Data Encryption Standard. Takes a 64-bit input and output. Effective key length is 56 bits. DES used to be standard
from 1977 till the late nineties. Sometimes DES is still applied in the form of triple-DES, where you apply DES three
times, using a different key every iteration. However, the second iteration uses a DES decrypt instead of encrypt.

### AES
Advanced Encryption Standard. Improvement on DES created in the early 2000s. AES features:
- Complete public design and algorithms;
- Key length of 128/192/256;
- Both hardware and software implementation possibilities.
