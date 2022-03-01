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

* https://tools.ietf.org/html/rfc9103 - DNS Zone Tr  ansfer over TLS
    * Many DNS servers do not permit zone transfers over DNS

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

- The lists of approved domains.
    - Very few sites use anything but *.com for their primary services.
          - .gov is often used for official, federal documents.
	  - The nationality domains are sometimes used for international services.
	  - Almost all bib ,cin domains are ignored, populated by domain
            squatters, or purchased by the .com domain owners to
            prevent domain squatting.

Vital RFCs
----------

- Valid IP addresses
    - All IPv4 addresses are 32 bits, all IPv6 addresses 64 bits.
    - Both have reserved, non-routable addresses which should never be permitted through a public router.
    - Both have overlapping spaces for loopback.
        - 127.0.0.1 is the IPv4 loopback, but all of 127.0.0/8 routes to it
	- ::1 is the the loopback. Enabling others involves theological arguments.

- Non-routable IP addresses
    - Almost everyone doing this uses 10.0.0.0/8
    - A few tools especially for home routers 192.168.0.0/16
    - Very, very few use 172.16.0.0/12
    - These leave plenty of space for local for business environments to provide local DNS records,
      and to split them up for useful departmental subdomans

- SOA records
    - These publish the owners of domains, the release number of
      updates, the duration of valid lookups, and the duration of
      failed lookups.
    - A great deal of software ignores these time values and the
      associated RFCs, "optimizing performance" by caching lookups
      for hours or even permanently.

- Subdomains
    - Any domain can have multiple subdomains.
      - "localdomain" can have "dev.localdomain", "prod.localdomain", or "mydepartment.localdomain"
    - A real subodomain normally has an SOA record, and NS records in its parent domain
      to track down or report failures if it is offline.
    - Simply calling a host "first.second.localdomain" does not create subdomains. Unless
      "second.localdomain" is its own domain with an SOA and NS records, the domain is still "localdomain",
      and the entry in that domain is "first.second"

- Wildcards
    - Wildcards have become very popular to route all DNS requests to specific addresses, especially
      for corporate addresses like "www.mycompany.com" and "wwww.mycomapny.com".
    - These work well, but can lead to chaos when '*.com' leads all erroneous DNS lookups ending in .com
      to be connected to Verisign's servers. This has happened with several domains.
        - The wildcard lookups are resolved before DNS suffixes are
          added, so "www.prod" under "www.prod.mycompany.com" wound up resolving to Verisign,
	  not the internala "www.prod.mycompany.com" hostname.

- Private domains
    - Private domains are not propagated to the Internet.
        - These must be selected carefully to avoid accidental conflict or
	  resolution as public addresses.
    - These are very broadly used for entirely internal services.

- DNSSEC
    - Securing corporate DNS services against fraudulent DNS records
      has become more importane.
    - Poisoned DNS records could and have connected people to entirely
      fraudulent DNS services.
    - DNSSEC helps prevent these replacements, but remains awkward and
      and combersome to enact for small environments.

- SPF
    - SPF records indicate what hostnames, IP addresses, or Domain
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

