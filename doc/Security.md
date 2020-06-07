# Security
Security deals with protecting a computer system from intentional misbehavior. This differs from random faults and
errors. While error codes can help you detect the latter, the former can still happen.

## Index
- [Ethical Hacking](#ethical-hacking)
- [Terminology](#terminology)
- [Attacker Model](#attacker-model)

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
