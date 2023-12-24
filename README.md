# bind9-resolver-example

Bind9 Selective Forwarding Resolver Configuration Example.

## Motivation

I recently came up with the idea to increase my knowledge regarding some of the
basic features which keep the internet running. Naturally, one of the first
things to pick up would be the Domain Name System (DNS) which, in my opinion, is
literally the backbone of everything we know as the modern internet today.

With some basic knowledge about how DNS works, I started with some basic Bind9
configuration which led me rather fast to a working recursive DNS resolver.
Being the person I am, I quickly decided that while this was fun, I did not
really learn anything about how to set up more complex scenarios. This led to
several hours thinking about a mode complex setup and ultimately ended with days
of intense configuration, troubleshooting and reconfiguration.

During this journey I encountered some rather interesting problems which brought
me to writing this example and providing my final configuration in this
repository.

One final think I want to address:
I am not an expert of DNS. Please do not use any of this code in your
environment if you do not know what you do (this is a general advice, you should
never use untested or code you do not understand).

## Scenario

The assumed scenario configured is as follows:

* We have three clients (located in different subnets and DNS-Domains)
* For simplification, each subnet has its own DNS-Domains and vice versa
* We have one central DNS forwarding resolver configured
* We have three authoritative DNS-Server providing the forwarding mapping zone
  data
* We host the following zones:
  * `foo.bar`
    Contains our three DNS-Server (`dns-srv-[01-03]`), one A RR for the DNS
    resolver (`dns-resolver-01`) and CNAMEs for our clients (`client-[01-03]`)
  * `a.foo.bar`
    The domain for our test client 1 (`dns-client-01`)
  * `b.foo.bar`
    The domain for our test client 2 (`dns-client-02`)
  * `c.foo.bar`
    The domain for our test client 3 (`dns-client-03`)
* We have one primary-secondary DNS-Server pair (`dns-srv-[01-02]`, providing
  authoritative answers for the zones `foo.bar`, `a.foo.bar` and `b.foo.bar`
* We have one primary DNS-Server (`dns-srv-03`) which provides authoritative
  answers for the zone `c.foo.bar`
* We have another recursive server already in place which we can forward any
  other DNS-Queries to
* Our existing setup is only part of a larger private network we do not know
  much about, so we have to forward queries for reverse mapping entries for
  private networks to the existing forwarder as well

I asigned the following IP addresses to the server/clients included in this
setup:

* `172.16.2.221` for `dns-resolver-01.foo.bar.`
* `172.16.2.251` for `dns-srv-01.foo.bar.`
* `172.16.2.252` for `dns-srv-02.foo.bar.`
* `172.16.2.253` for `dns-srv-03.foo.bar.`
* `172.16.10.10` for `dns-client-01.a.foo.bar.`
* `172.16.10.221` for `dns-resolver-01.a.foo.bar.`
* `172.16.11.10` for `dns-client-02.b.foo.bar.`
* `172.16.11.221` for `dns-resolver-01.b.foo.bar.`
* `172.16.12.10` for `dns-client-03.c.foo.bar.`
* `172.16.12.221` for `dns-resolver-01.c.foo.bar.`
* `10.0.0.250` for the upstream recursive DNS resolver

Putting everything together, the scenario looks something like this:
![Scenario Sketch](scenario_sketch.png)
