---
permalink: /homelab/software/dns
title: DNS

sidebar:
    nav: homelab-software-nav

classes: wide
---

Domain name system (DNS) is responsible for translating human-readable domain names (ie Olivando.me) to IP addresses which computers are able to use to locate and connect to other devices.

DNS relies on four kinds of servers to properly function. Two of which are used in my homelab:

### Resolver name server
Resolver name servers are responsible for finding answers for DNS queries. They can do this by either (a) Responding directly to the query using a DNS record they have cached from a previous query (b) forwarding the query to the next Resolver name server in the DNS chain as described by their configuration.

**For example:** within my Homelab I have my Resolver name servers configured to route all traffic to 8.8.8.8 (Google's DNS server's IP) if it does not have the DNS record cached.
{: .notice--info}

### Authoritative name server
An Authoritative name server is responsible for providing definitive answers to DNS queries, meaning unlike resolver name servers it does not rely on cached responses from other name servers and instead will provide answers directly from it's domain zone configurations.

Authoritative name servers fit into the DNS chain using the configuration on Resolver name severs. This allows manual configuration of routing depending on the domain name in the request.

**For example:** within my Homelab I have my Resolver name servers configured to route all traffic for home.olivando.me to my authoritative name servers which are configured with DNS records for the home.olivando.me DNS zone.
{: .notice--info}