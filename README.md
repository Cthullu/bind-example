# bind9-resolver-example

Bind9 Selective Forwarding Resolver Configuration Example.

---

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

---

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
* We do not have to consider reverse mapping zone delegation
* We have another recursive server already in place which we can forward any
  other DNS-Queries to
* Our existing setup is only part of a larger private network we do not know
  much about, so we have to forward queries for reverse mapping entries for
  private networks to the existing forwarder as well

For simplification, I decided to use separate reverse mapping zones for each
forward mapping zone:

* `172.16.2.0/24` - `foo.bar`
* `172.16.10.0/24` - `a.foo.bar`
* `172.16.11.0/24` - `b.foo.bar`
* `172.16.12.0/24` - `c.foo.bar`

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

---

## Configuration

Before I jump into the configuration (and this is were I will also point to some
of the challenges I faced):

* I used Ubuntu 22.04
* I used the Bind9 version provided by the system package manager (`9.18.20`)
* I discovered quiete early, that I like to have my configuration files for all
  of my servers organized in a specific way which differs from the system
  default
  * All configuration files life inside `/etc/named/` or a subfolder
  * All zone files life inside `/etc/named/zones`
  * I use the main `named.conf` file only to include other configuration
    snippets
  * All configuration snippets life inside `/etc/named/conf.d`
  * Transfered zones, logs etc. life at `/var/named` or a subfolder
* I always configured rndc to manage my server
  * Configuration file lifes at `/etc/named/`
  * Configuration files for rndc are not included in this repository
* I adapted the systemd service file
  * To not check for configured systemd
  * To not load any external environment file

I configured the server in the following order:

* Primary name server for `foo.bar`, `a.foo.bar` and `b.foo.bar` (`dns-srv-01`)
* Secondary name server for `foo.bar`, `a.foo.bar` and `b.foo.bar`
  (`dns-srv-02`)
* Primary name server for `c.foo.bar` (`dns-srv-03`)
* Forwarding resolver (`dns-resolver-01`)
* Clients (`dns-client-[01-03]`)

---

### dns-srv-01

### dns-srv-02

### dns-srv-03

### dns-resolver-02

---

## Conclusion

After I finished troubleshooting, created all of the configuration provided by
this repository and performed a lot of tests, the last part of learning
something is always to draw a conclusion.

One thing I did take with me during this learning experience was a lot of
knowledge about the various roles a (simple) Bind9 server can take. This makes
it an incredible tool which can be a beast to first understand. However, once
the conceptional idea is clear, it becomes a lot more natural to configure the
various parts.

I also learned a lot about (selective) forwarding of queries, recursive and
iterative queries and once again was reminded, how important a good logging
configuration is to ease troubleshooting.

The final thing I took with me is, that while the documentation of Bind9 from
ISC at [readthedocs.io][1] is already great, it does not cover all the
possibilities which can be performed by using Bind9 (which is most likely due to
the vast amount of confgiuration scenarios). Thankfully, the internet is full of
confgiuration references, ideas and configuration snippets which helped me a lot
during this project. The hardn part is simply to find and put those peaces
together.

[1]: https://bind9.readthedocs.io/en/latest/
