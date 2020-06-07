# Transport Layer
The application layer uses the layers below it to create functionality in applications. Examples are email, DNS, HTTP
and more.

## Index
- [DNS](#dns)
- [Email](#email)
- [WWW](#www)
- [CDN](#cdn)

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

### Email Protocols
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

#### SMTP
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

#### IMAP
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

## CDN
*Content Delivery Networks* can be used to deliver content at one domain name, from multiple physical machines. When you
need to serve a lot of (static) data to many users, to the point where server farms are not enough, CDNs are ideal. They
allow you to scale globally, meaning you can have data centers all over the world to ensure low latency content delivery
to all your users.

CDNs completely turn the traditional web caching on its head - instead of having users look for a copy of the requested
resource or page in a nearby cache, the provider itself places such a copy somewhere close. CDNs follow a tree structure
with its root being the *origin server*. The origin server is the server that generates or stores the original content.
Its children are CDN nodes, which own a copy of the original content, which they can then serve to clients connected to
them. Since CDNs form a tree structure, it is very easy to scale up. You can simply add a sibling CDN node in a region
when that respective region's node is often at maximum load.

If your origin server can only handle so much bandwidth, you can fix that by simply creating CDN nodes as children of
other CDN nodes. This increases the height of the tree, and ensures that the added CDN node don't interact with the
origin server at all, instead their parent CDN node does that. You can repeat the exact same process over and over again
to increase the size of your tree while maintaining a well-working network. It is of course important to ensure your
tree is as efficient as possible to reduce cost.

Note that CDNs don't only reduce latency because they are often nearer to you than the origin server; they can also
improve bandwidth. This is not only because CDNs handle a small partition of the total amount of clients, but also
because a lower latency allows TCP slow-start to ramp up more quickly, which means you will be transferring at maximum
bandwidth sooner.

### Organizing Clients
While the distribution tree is rather simple, it is less simple to have client connect to the right node, as you have
just one outwards-facing domain name. A possible solution to this is using Web proxies to make clients able to connect
to their nearest CDN node. In practice, though, this solution falls short for a few reasons:
1. Clients in a given part of the network probably belong to different organizations, so they are probably using
different web proxies;
2. There can be multiple CDNs, but each client only uses a single proxy cache - which CDN should the client use?;
3. Proxies are configured by clients, which pretty much makes this idea suck as no non-tech person would configure a
proxy just to use Twitter.

### Mirroring
A better solution can be found in *mirroring*, where the origin server's content contains explicit links to different
mirrors (which are just the CDN nodes). This makes it easy for users to manually select a nearby mirror. You probably
have visited a website before that asks you your location the moment you enter their website, and once you select your
country it redirects you to `nl.website.com`, for instance. This is exactly what mirroring is. It is simple and
practical as it is essentially just having different websites for different regions. You can simply set it up in your
DNS by having a subdomain like the previous one simply point to a mirror's IP address.

Mirroring is also often applied when downloading software from repositories. If you install Ubuntu, it will ask you for
your location. It will then select which repositories to download stuff from based on that location. Check
`/etc/apt/sources.list` if you don't believe me, they will probably contain `http://nl.archive.ubuntu.com/ubuntu/`.

You can generally only serve static content with this solution, as dynamic content usually requires a database. If you
have one central database, your CDN will be slow. If you distribute your database, it will be either non consistent, or
not available. Not available would once again slow your CDN down, and non consistent is bad. As you can see the moment
you step out of the realm of static content a lot of issues arise. In some cases non consistency would probably be fine,
though, as you might only be using the local copy of the database. If you ensure your databases are consistent within
the time it takes to physically travel from one CDN region to another, it would function just fine. There's a lot of
if's here though - let's leave this one to the professional network guys.

The downside of mirroring is that it requires users to do the distribution.

### DNS Redirection
Another approach, which overcomes the difficulties of mirroring, is *DNS redirection*. As previously discussed, your DNS
pretty much transforms a domain to an IP address, however, most name servers won't know the domain and will redirect you
to another name server instead. This means that you can set up your own name server that dynamically determines the IP
it resolves when you query their domain name. You can now apply mirroring just like before, but without the user
noticing or having to do anything! This solution is a bit more expensive, though. A disadvantage is that other name
servers can mess your distribution up by resolving your domain name statically. So technically you could throttle an
international website by redirecting all your local traffic to a distant CDN node.

### Nearest Node
We now know how CDNs work, but not how a certain node is chosen (when not done manually). Defining the nearest node is
not a question of which node is geographically near, but rather which is nearer on the internet itself, i.e. which node
has the shortest route. This is the *network distance*. Another factor is the load that is already being carried by the
CDN node. If you happen to live next to a Google CDN, but everybody happens to be googling at the moment, you might
connect to a Google CDN in the next city instead (where nobody is googling). So balancing load is also important when
choosing which CDN node is ideal for a client.

The techniques for using DNS for content distribution were pioneered by Amakai, starting in 1998. If you've ever used
Steam you might have noticed that all of their [images](https://steamcommunity-a.akamaihd.net/economy/image/-9a81dlWLwJ2UUGcVs_nsVtzdOEdtWwKGZZLQHTxDZ7I56KU0Zwwo4NUX4oFJZEHLbXH5ApeO4YmlhxYQknCRvCo04DEVlxkKgpot621FAR17P7NdTRH-t26q4SZlvD7PYTQgXtu5cB1g_zMyoD0mlOx5UM5ZWClcYCUdgU3Z1rQ_FK-xezngZO46MzOziQ1vSMmtCmIyxfkgx5SLrs4SgJFJKs/360fx360f)
are hosted using Amakai. Amakai was the first major CDN and became the industry leader, so no surprise that a platform
with a lot of data transfer such as Steam uses their services. Since 1998 more companies have gotten into this business,
so it's now a competitive industry with multiple providers. Most companies/websites that use CDNs hire firms like Amakai
to handle it for them. They give the firm their content, which they then push to CDN nodes. The owner then rewrites any
of its web pages that link to the content (e.g. your <img> src is now the CDN node instead of your own web server).
Since you usually hire firms like Amakai for your CDN needs, it's hard to serve dynamic content as it requires dedicated
software.

Another advantage for sites to use a shared CDN is that future traffic is hard to demand. Surges in demand - called
*flash crowds*, can easily overflow or throttle a site's web servers. An example is the Florida Secretary of State's
website, which is usually in high demand on the election day in the US. On November 7th, 2000, it suddenly became one of
the busiest websites in the world as people wanted to check the election result. As a result the site crashed. If it had
been using a CDN this could have been prevented. 

For CDN nodes to have optimal connectivity they can be placed at ISPs. This is beneficial for everyone; the website will
load faster, which makes the company money, makes the ISP look good, and delivers content to the user faster. ISPs
don't have to pay for this, so it's a no-brainer for them.
