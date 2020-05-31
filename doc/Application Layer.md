# Transport Layer
The application layer uses the layers below it to create functionality in applications. Examples are email, DNS, HTTP
and more.

## Index
- [DNS](#dns)
- [Email](#email)
- [WWW](#www)

## DNS
DNS is quite literally what the initialism stands for: domain name system. The system allows us to enter a domain name
and end up connecting to an IP address. This is not only handy because remembering IP addresses is difficult, it is
also beneficial because IP addresses can change. But how is this done?

The DNS name space has a hierarchical structure. Top level domains are the parts after the dot in most urls (.nl, .com).
Many top level domains belong to countries, but there are also quite some generic ones (.com, .edu, .museum, .org). Top
level domains are controlled by *ICANN* (Internet Corporation for Assigned Names and Numbers) and have **subdomains**.
You can request second-level domains (`google.com,`, `google.nl.`, `tudelft.nl.`) from **registrars**. If you have your
own second-level domain, you can create even more subdomains below the second-level domain you own. Examples could be
`en.google.com.` for the English version or `nl.google.com.` for the Dutch version of Google.

Some countries handle subdomains a bit differently. For example, in the UK the subdomain .uk is not used. Instead, you
use second-level domains such as `.co.uk.` for commercial websites, `.ac.uk.` for academic websites, etc.

### Name Servers
Name servers are where the magic of DNS happens - they route second-level domains to IPs affiliated to that second-level
domain. The location of name servers is configured (by default) via *DHCP*. Your operating system keeps track of name
servers and dynamically selects which to use. You can of course override this, though.

### Recursive Queries
When you query a (local) name server for a certain domain, that name server might not have the information you are
querying. If that's the case, that name server will request this information from other name servers. First off the
*root name server* is asked where the name server of a specific top-level domain is (eg `.nl`). The top-level domain
name server is then asked where the name server of the second-level domain is, which will then be queried for the IP.

In actuality, each contacted name server is asked where the full domain name leads to, but usually they don't know.

In practice DNS queries don't work quite like this, as both computers and name servers usually do quite a bit of
caching.

### Iterative Queries
Iterative queries are similar to recursive queries, but instead of the local name server handling communication with
other, your computer handles it. Your local name server will give you the address of the root name server, which will
once again give you the address of another name server until you get to a name server that knows where you should go.

A set up like this is handy because you don't need a local name server. Recursive queries are better if your machine is
not very powerful, though.

### Domain Resource Records
When queried, name servers respond with *domain resource records*. These records can contain:
1. IPv4 address (record type A);
2. IPv6 address (record type AAAA);
3. Domain that accepts email (record type MX);
4. Name server for this domain (record type NS);
5. And so on....  

## Email
You can send and receive email on your own domain, otherwise you can of course also sign up for free email services,
like Gmail, Outlook, etc.

Email messages consist of multiple parts:
1. An envelope;
2. A header;
3. A body.

### The Envelope
The envelope is something your message ends up in. It generally consists of the sender's email address, the recipients
\email address, whether the message
has been encrypted, etc. The envelope is used by mail servers to ensure your email ends up where it should end up. It is
pretty much exactly what a
physical envelope is, and functions the same way; it tells the postal service where the message should go.

### The Header
Unlike the envelope, the header is part of the message - it is intended for the user, not the mail server. It can
contain information about the sender and
recipient, such as the name of who sent/received the email. It can also contain a subject.

### The Body
The body is exactly what you would expect it to be; the data that has been sent.

## Email Protocols
Email uses multiple protocols:
1. **POP3** and **IMAP** are used to allow users to interact with their mailbox;
    - POP3 is older and generally deletes messages once you read them - not handy in the current era where everyone has
    at least 2 devices.
2. Users and *Message Transfer Agents* use **SMTP** to send email from source to destination.

Generally, when you send an email, it will be sent to your email's message transfer agent using SMTP (+ authentication
and possibly other extension).
This server will then forward it to the recipient's message transfer agent using SMTP, where the recipient can fetch
their mail using POP3 or IMAP.

But what if your message transfer agent doesn't know where the recipient's server is? This is where DNS saves the day
once again. You can simply query
a nameserver to get an IP which your message agent will include when using SMTP.

### SMTP
SMTP uses ASCII, which is not ideal as you can't really send binary data with ASCII. Another huge disadvantage of basic
SMTP is that it does not include authentication, meaning you can easily forge the `from` field to send emails in someone
else's name. Luckily many SMTP extensions address these issues.

#### Multipurpose Internet Mail Extensions (MIME)
Generally you want to send more than just text, which is what MIME allows you to do. It introduces additional headers
such as Content-Type (just like in HTTP!), to indicate which type of data the message is. Examples are:
- text/plain;
- text/html;
- image/jpeg;
- image/gif;
- video/mp4;
- video/mpeg.

These headers can be used to create messages with fancy HTML, or to attach attachments.

When MIME was first introduced, servers were not expecting non-ASCII data. Since you can't just tell the entire internet
to update their servers, something compatible with ASCII-only servers had to be made. The solution to this is somehow
encoding binary streams into ASCII. Base64 is ideal for this as its alphabet consists of the group [A-Za-z0-9+/].

With base64, every 6 bits are translated into 1 character. This means that instead of byte-wise you are now in the 6-bit
wise realm. It doesn't really matter though, as servers that can do MIME types will simply decode the base64 back to
the original binary stream.

A huge disadvantage is that ASCII uses 7 bits per character, while base64 uses 6. This means that every 6 bits will take
up one additional bit when encoding a binary stream as base64. If you're encoding ASCII, this is even worse. Since
everything is generally byte-wise, a 0-bit is prepended onto the 7-bit ASCII, which makes it even less efficient. All in
all ASCII encoded as base64 causes an additional bit for every 3 bits.

Another (small) disadvantage is the need to pad to ensure we have a stream with a length divisible by 6. This padding is
always an equals sign (=), you have probably seen it before in many places.

### IMAP
IMAP is probably what you use to communicate with your mail server. It is a protocol that allows you to manipulate
mailboxes and replaced POP3. There are a few common commands:
1. LOGIN - log in to your mail server;
2. FETCH - fetch messages from a folder;
3. CREATE - create a folder;
4. DELETE - delete a folder;
5. EXPUNGE - remove messages marked for deletion.

Note that web mail clients generally don't use IMAP to communicate with mail servers, but instead opt for HTTP(s) using
APIs.

## WWW
The world wide web's main purpose is having data available even if it's not near you or on your local network. It all
started at CERN. Thw World Wide Web Consortium (W3C) is an organization devoted to:
1. Developing the web;
2. Standardizing protocols;
3. Improving compatibility between sites.

The web uses HTTP for requesting and delivering resources. HTTP is built on top of TCP/IP to ensure consistency. Since
making a new connection for each request is quite inefficient, connections are generally **persistent**. This means that
your browser will issue multiple requests over the same TCP connection, greatly reducing the overhead of fetching
resources.

You can also apply **pipelining** to speed up things, which is not really what you would think it is. It just means
being able to have an outgoing request while the response to your previous request has not been fully received yet, on
the same connection.

Now, of course DNS comes up again! Domain names are used in **URLs** (*Universal Resource Locators*) to be able to
target specific resources. URLs generally consist of a protocol, domain name, and path (a lot more, actually, but you
can look that up yourself). You can also include a port in your URL, but since that is not something you'd expect users
to understand, by default port 80 (for HTTP) and port 443 (for HTTPS) are usually used.

Originally browsers were mainly text-based, but with the invention of MIME types this all changed. Just like in email
headers, HTTP headers also have a Content-Type field that can be used to indicate data types.

There are generally two ways of dealing with MIME types. One of them is having your browser take care of it, and the
other is having your browser let another program take care of it. So, the former could mean your browser has a plugin
that allows it to play videos, while the latter means it opens whatever your default video player is and plays
downloaded videos there.