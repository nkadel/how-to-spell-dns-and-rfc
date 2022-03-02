How to Spell DNS and RFC
========================

I'm an old technology fan, and therefore not deeply invested in
PowerPoint, Microsoft Word, and various "let's use another color and
typeface to layer in more meaning font ware". Keep it Simple, Stupid,
is very useful for software that will be read or used by other people.

DNS RFCs
--------

There are dozens. Too many for most of us to follow in every detail.

* https://www.statdns.com/rfc/

IANA relation to DNS
--------------------

The IANA sets what are valid IP addresses, and who is granted control
over them.  Since DNS is specifically turning hostnames into IP
addresses, and some vice versa, the IANA's RFCs are like laws of
physics for DNS

There are confusing parts of it.
    - 127.0.0.2, for example, is part of the loopback addresses space,
      and will always hit the "localhost" as well as 127.0.0.1.
    - 2105834626 is 127.0.0.1 written as a decimal number. 


Irrelevant RFCs
---------------

Ironically, many DNS RFCs are irrelevant to day to day operations for nearly all of us.

* https://tools.ietf.org/html/rfc9103 - DNS Zone Transfer over TLS
    * Many DNS servers do not permit zone transfers of DNS zones.
    - Most clustered DNS servers have some other cluster backend.
    - It can be incredibly useful for reviewing zones for dangling
      entries, but Samba for example doesn't support it at all.


* All the reverse DNS RFCs
    - Many sites do not support reverse DNS at all.
    - Others use NAT or outward facing proxies to conceal their
      internal DNS, and non-routable IP addresses, so reverse DNS
      means nothing to anyone else.
    - Those exposed IP addresses are often transient for cloud services,
      or owned by the ISP or cloud service, not the corporation hosting
      services behind them.
    - Reverse DNS is most useful for internal DNS and connection logging.
        - Many of these suffer from siloing and design schizophrenia.
	- Many of these suffer from poor synchronizaiton and very poor
	  consistency with other DNS.
    - AD is particularly bad about these.
        - AD DNS permits multiple PTR records.
	- Poor or inconsistent cleanups encourage leaving erroneous PTR.

* All the IPv6 RFCs
    * Many if not most sites simply use IPv4 internally with
      non-routable addresses, and oblige their exposed services to use
      proxies or some distinct outward facing gateway.
    - This is true for most home routers as well.
    - IPv6 is useful to expose every device in a large environment to
      the Internet, but relies on intelligent firewalls to protect them.

* The lists of approved domains.
    - Very few sites use anything but *.com for their primary services.
          - .gov is often used for official, federal documents.
	  - The nationality domains are occaionally used to segregate
	    .com from from international services
	  - Almost all bib ,cin domains are ignored, populated by domain
            squatters, or purchased by the .com domain owners to
            prevent domain squatting.

Vital RFCs
----------

* Valid IP addresses
    - All IPv4 addresses are 32 bits, all IPv6 addresses 64 bits.
    - Both have reserved, non-routable addresses which should never be permitted through a public router.
    - Both have overlapping spaces for loopback.
        - 127.0.0.1 is the IPv4 loopback, but all of 127.0.0/8 routes to it
	- ::1 is the IPv6 loopback.

* Non-routable IP addresses
    - Almost everyone doing this uses 10.0.0.0/8
    - A few tools especially for home routers 192.168.0.0/16
    - Very, very few use 172.16.0.0/12
    - Non-routable addresses are often served by private or
      internal only DNS.
    - These leave plenty of space for local for business environments to provide local DNS records,
      and to split them up for useful departmental subdomans

* SOA records
    - These publish the owners of domains, the release number of
      updates, the duration of valid lookups, and the duration of
      failed lookups.
    - A great deal of software ignores these time values and the
      associated RFCs, "optimizing performance" by caching lookups
      for hours or even permanently.

* A records
    - A records tie hostnames to IP addresses.
    - A records are not necessarily unique. One hostname may be
      linked to many IP addresses, and in large environments often are.
    - Which IP address an A record leads to is often a crapshoot,
      baswed on DNS views, local routing, application and system DNS
      caching lookups, stashed faled lookups and the vagaries of your
      TCP stack.
    - Some applications cache such records at start time and require a
      restart to change them. And yes, postfix did this.

* CNAME records
    - CNAMEs are extremely useful ways to delegate a DNS hostname to be
      resolved with another DNS record.
    - The chain can be very long, and very confusing, especially with DNS
      servers that do regionalized lookups and other forms of split views.
    - CNAMEs involve a second DNS lookup, which takes a modest amount of time
      and resources, but is vital to web proxies around the world.
    - Many of these web proxies serve multiple websites, and sort out the
      traffic based on the hostname in the HTTP request.
    - Multiple CNAME records are forbidden by various RFCs, though
      some DNS servers do permit them.
          - This way lies madness.

* Multiple A records
    - Multiple hostnames may link to the same target address or addresses.
    - Multiple A records are useful for various forms of high availability, such
      as multiple AD domain controllers.
    - It is useful for debugging and logging if each IP also has its
      own unique hostname, with a unique and matching PTR associated with it.
    - Very few systems do that multiple, unique hostname and multiple A records
      automatically.

* TXT records
    - Very handy for storing data about a domain.
    - Used to be used for SPF, can still hold SPF data.

* MX records
    - Ranked list of servers where email for the
      domain is delivered.

* RR records
    - RR records for SPF indicate hostnames, IP addresses, or Domain
      Keys are permitted to send email from a particular DNS record.
      That DNS record gets the email bounces
    - SPF was originally a pure TXT record, set by the owner of the
      DNS domain, to validate hostnames and IP addresses in the SMTP
      defined "From ' address, the address that gets email bounces.
      That field is set by the SMTP server, and is not so easily
      forged as a 'From:' field
    - The SPF RFC was combined with "Domain Keys" from Microsoft,
      which have nothing to do with the original SPF and requires
      keys purchased from Microsoft.
    - SPF has issues with mail forwarding.

* Subdomains
    - Any domain can have multiple subdomains.
      - "localdomain" can have "dev.localdomain", "prod.localdomain", or "mydepartment.localdomain"
    - A real subodomain normally has an SOA record, and NS records in its parent domain
      to track down or report failures if it is offline.
    - Simply calling a host "first.second.localdomain" does not create subdomains. Unless
      "second.localdomain" is its own domain with an SOA and NS records, the domain is still "localdomain",
      and the entry in that domain is "first.second"

* Wildcards
    - Wildcards have become very popular to route all DNS requests to specific addresses, especially
      for corporate addresses like "www.mycompany.com" and "wwww.mycomapny.com".
    - These work well, but can lead to chaos when '*.com' leads all erroneous DNS lookups ending in .com
      to be connected to Verisign's servers. This has happened with several domains.
        - The wildcard lookups are resolved before DNS suffixes are
          added, so "www.prod" under "www.prod.mycompany.com" wound up resolving to Verisign,
	  not the internala "www.prod.mycompany.com" hostname.

* Private domains
    - Private domains are not propagated to the Internet.
        - These must be selected carefully to avoid accidental conflict or
	  resolution as public addresses.
    - These are very broadly used for entirely internal services.

* Split views
    - Split zones make it possible tell a DNS servers that queries from one address space
      resolve one way, and from another space resolve another way.
    - Split views are especially common for websites that have different internal and
      external load balancers.
    - Split views are often very fragile, with different TTLs, poor synchronization,
      and very confusing behavior as routing tables, VPNs, and proxies re re-arranged.
    - Split views are the core of high-availability, and localizing
      proxies such as Akamai and Cloudflare.


* SRV records
    - Service records for Active Directory.
    - AD and its SRV records are their own subject, and have almost nothing
      to do with Internet facing services.

* DNSSEC
    - Securing corporate DNS services against fraudulent DNS records
      has become more importane.
    - Poisoned DNS records could and have connected people to entirely
      fraudulent DNS services.
    - DNSSEC helps prevent these replacements, but remains awkward and
      and combersome to enact for small environments.

Not clear in RFC
----------------

* /etc/hosts predates and overrulesmost RFCs and DNS

Listing the hosts, and IP addresses, in a local /etc/hosts always precedes
DNS. (Well, OK, that can be reset in nsswitch.conf, but no one does.)

    - Many hosts ignore DNS in favor of local tuning of /etc/hosts.
    - These can accumulate startling errors, especially when host images
      are duplicated carelessly or hostnames are altered carelessly.
    - Never, never, never use both the hostname and the localhost
      on the same line. Split them if needed.
      127.0.0.1 localhost.localdomin localhost localhost4.localdomain localhost
      ::1 localhost.localdomin localhost localhost64.localdomain localhost6
      127.0.0.1 hostname.domainname hostname # pick any or all of these
      ::1 hostname.domainname hostname
      a.b.c.d  hostname.domainname hostname
      
* Fake DNS aliases
    - Some systems, such as modern Active Directory with its "aliases" and
      Infoblox with its "Host" records, store back end information
      linked to specific hosts with addintional IP addresses linked
      to the same hostname and its primary IP address.
    - These are not DNS aliases, aka CNAMEs, they are back end generated
      multiple A records, and no RFC defines them as "aliases".
    - This is the sort of "let's invent words" which make RFCs vital
      for clear communication.
    - These fake aliases can be very awkward to track down or clean up
      as hosts expire or are migrated.

Emergent behavior
-----------------

* Ring around the reverse DNS
    - Some tools, like sshd, look up both forward and reverse DNS records
    - PTR of IP address
      - Hostname reported by PTR
      - PTR of IP associated with hostname
    - Thhis leads to single threaded musical chairs with DNS servers,
      until and unless those records match.
    - This can be disabled by setting the sshd to start with the '-u0'
      - The sshd_config options do not modify this behavior

* Domain squatters
    - IANA deliberately fosters domain squatting
    - Many DNS providers log failed DNS requests and immediately
      reserve the failed domain themselves
    - Squatters lurk and claim expired domains withing moments
      preying on the slow or delayed DNS admins of the world.
