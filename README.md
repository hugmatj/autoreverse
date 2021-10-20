# autoreverse
A no muss, no fuss ipv4 and ipv6 reverse queries DNS name server

`autoreverse` is a simple authoritative DNS server with the solitary goal of making it
trivially easy to auto-answer reverse queries for ipv4 and ipv6 with minimal configuration.

In some cases you'll be able to deploy `autoreverse` without *any* configuration at all,
and in many cases you'll be able to deploy `autoreverse` with local requirements expressed
in just a few command-line options.

The motivation for `autoreverse` is to avoid *ever* having to look at, yet alone edit,
another **ip6.arpa** zone file (or it's elder sibling in-addr.arpa) and thus avoid having to deal
with reverse nibbles, inscruitable delegation boundaries and mind-numbing hexdecimal
strings.

In short, you should consider `autoreverse` as pain relief from reverse zone management.

### Project Status

[![Build Status](https://travis-ci.org/markdingo/autoreverse.svg?branch=master)](https://travis-ci.org/markdingo/autoreverse)
[![Go Report Card](https://goreportcard.com/badge/github.com/markdingo/autoreverse)](https://goreportcard.com/report/github.com/markdingo/autoreverse)
[![codecov](https://codecov.io/gh/markdingo/autoreverse/branch/master/graph/badge.svg)](https://codecov.io/gh/markdingo/autoreverse)

### Key Features

`autoreverse` responds to PTR queries. That's it. That's all it does. Oh, it can
synthesize responses and also infer responses if you choose to provide forward zone
details. Still, it only responds to PTR queries, just in an easy and automatic way.

(For those new to DNS, a "reverse query" and "reverse lookup" are shorthand for a PTR
query in the "reverse DNS tree". These terms are used interchangeable in this document. If
you wish to know more, [Wikipedia](https://en.wikipedia.org/wiki/Reverse_DNS_lookup)
has details.)

### What *doesn't* autoreverse do?

Everything that a *real* name server does. `autoreverse` doesn't serve address records or
delegate zones. Importantly, `autoreverse` is **not** designed to support high-volume
query rates. It's preferred resting spot is on the edge of the Internet serving a modest
query rate.

### Who should use autoreverse?

`autoreverse` is intended for small installations and home-gamers who want the reverse
lookup of their IP assignments to say something useful. Most often this occurs in
conjunction with ISPs who allow name server delegation of customer assigned
addresses. In most cases of course, responses to reverse lookups are used purely for
aesthetic or logging purposes, so deploying `autoreverse` rarely has any operational
consequences; positive or negative.

Having said that, it is rather nice to see recognizable names show up in connection
reports rather than some inscrutable stream of truncated hexadecimal. If you agree, you're
in the right place.

That's not to say `autoreverse` can't be deployed in other scenarios. You might be an
sysadmin who wants all reverse queries directed to a zero-maintenance
system. `autoreverse` can probably take on that task.


### What do you mean by "minimal configuration"?

`autoreverse` deduces just about everything possible. That means `autoreverse` can start
up and respond to PTR queries with the following invocation:

```sh
# /usr/local/sbin/autoreverse
```

Here's a slightly more complex invocation which overrides some defaults and deduced
values:

```sh
# /usr/local/sbin/autoreverse -origin example.com -listen 198.51.100.1
```

Finally, if you want to incorporate your own forward names into the answers such an
invocation might look like:


```sh
# /usr/local/sbin/autoreverse -listen 2001:db8::1 /etc/nsd/example.net.zone
```

In all cases you should notice a complete absence of any tell-tale signs of reverse zone
files or PTR records. That's on purpose of course. Truth be told, `autoreverse` doesn't
even know how to parse PTR records; it only knows how to generate them!

More complicated invocations are possible so it's worth scanning the
[manpage](./autogen/MANPAGE.txt) if you plan to deploy `autoreverse`. But hopefully you're
left with the impression that running `autoreverse` is straightforward.

## Principles of Operation

The `autoreverse` DNS server mostly relies on deduction and synthesis to minimize or
eliminate configuration and operational settings. It also relies on a number of
aspects of the DNS to function correctly:

1. The DNS delegation tree means that name servers generally only get queries for zones
they are responsible for. That is, within their bailiwick.
2. Out of necessity, traditional name servers typically refuse to respond to queries outside their
pre-configured bailiwick; but this necessity does not always hold for reverse queries.
3. PTR responses are mostly for aesthetic or rendering purposes; they rarely have any
operational impact.
4. A well-established convention, particularly for ipv6, is to synthesize PTR responses
based on the qName.
5. No one cares if a name server responds to queries `outside` their
bailiwick. It may be non-standard behaviour, but it's mostly harmless. And it's especially
harmless in the case of PTR queries.

All of which is a long-winded way of saying that, by default, `autoreverse` auto-answers
any well-formed PTR query it receives.

Due to delegation constraints `autoreverse` only gets legitimate queries within its
bailwick anyway, thus the only time `autoreverse` responds with an out-of-bailwick answer
is due to rogue or exploratory queries. And who cares if these queries get an unexpected
answer? Thus `autoreverse` takes the reasonable approach that it's only going to get asked
a question it's meant to answer. And so it does. Of course it doesn't need a configuration
file to tell it that!

To state this approach another way, `autoreverse` assumes it can answer every well-formed
PTR query it gets *unless* it's specifically constrained.

## Installation

`autoreverse` should compile and run on most Unix-like systems. It *should* also run on
Windows.


### Prerequisites

To compile `autoreverse` you need a recent version of [go](https://golang.org). 1.17 or
later is recommended. For security reasons, avoid compiling with anything prior to 1.16.2
on Linux targets due to possible issues with Setuid() and threads.

### Target Systems and cross-compiling

`autoreverse` has been tested on various CPU architectures with FreeBSD, Linux and
macOS. The [Makefile](./Makefile) in the installation directory builds and installs
`autoreverse` into `/usr/local/sbin`. Modify as necessary.

`autoreverse` *may* compile and run on Windows but you can also cross-compile to Windows
on a Unix-like system. To assist in this the Makefile contains the `windowsamd64` and
`windows386` targets.

Perhaps of most interest to residential deployments is the possibility of installing
`autoreverse` on your gateway router. To that end, the Makefile has targets for a
few *prosumer* routers such as Ubiquiti Edge Routers and Mikrotik Router Boards. It should
be possible to target other platforms too! This project is very interested to hear of
attempts to install `autoreverse` on gateway routers so please provide feedback of
successes *and* failures.

### Fetch and Make

To fetch, compile and install `autoreverse`, run the following commands:

```sh
go get -d -u github.com/markdingo/autoreverse     # Ignore the warning about no go programs

cd $GOPATH/src/github.com/markdingo/autoreverse

make clean all             # Compile everything
sudo make install          # Installs into /usr/local/sbin
```

If everything went well, you are now ready to run `autoreverse`.

## Getting Started

Note: This section is here mainly to provide guidance to relatively novice admins. If this
text seems confusing then it might be best to consult with someone more experienced at
deploying name servers. Experienced admin can probably skip this section entirely.

There are three steps to getting `autoreverse` answering reverse queries.

The first step is to select a system with at least one publicly visible IP address
(preferably an ipv4 *and* an ipv6 address) which can run `autoreverse` and accept DNS
queries from the global Internet. In small installations that might be on the gateway
router or possibly an internal system with port 53 forwarded from the router for ipv4.

The second step is to run `autoreverse` on the selected system/IP(s). By default
`autoreverse` needs to run as root to listen to port 53 traffic, however, if you are port
forwarding it's not unreasonable to port forward ipv4 to a non-privileged port and avoid
the need to run as root. (The specifics of how you run `autoreverse` and have it restart
on failure and system reboots is beyond the scope of this document.)

Once `autoreverse` is running, it's worth issuing a few queries to make sure `autoreverse`
is functioning prior to proceeding to the final step. The `dig` command is useful for
this; try these commands:

```sh
dig @IP-Address -x 198.61.100.1
dig @IP-Address -x 2001:db8::1
```

Where IP-Address is the publicly visible IP address you selected previously.

Assuming all is well, you will get a recognizable PTR response to both of these queries.

Which leaves the third and final step; arranging the reverse delegation to your selected
IP(s). Again, how this is done varies widely amongst ISPs and is obviously beyond the
scope of this document. Once delegation is complete you should be able to verify the
path down to your `autoreverse` instance with a regular resolving query, such as:

```sh
dig -x one-of-your-ip-addresses
```

and see a query that is recognizably an answer from your instance of `autoreverse`. If
enabled, you should also see the query show up in the `autoreverse` log.

And that's it! No more hex nibbles for you.

### Community

If you have any problems using `autoreverse` or suggestions on how it can do a better job,
don't hesitate to create an [issue](https://github.com/markdingo/autoreverse/issues).
This package can only improve with your feedback.

### Copyright and License

`autoreverse` is Copyright :copyright: 2021 Mark Delany. This software is licensed under
the BSD 2-Clause "Simplified" License.
