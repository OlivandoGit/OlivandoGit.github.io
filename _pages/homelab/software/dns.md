---
permalink: /homelab/software/dns
title: DNS

sidebar:
    nav: homelab-software-nav

classes: wide
---

**Please be advised:** This page is still in the draft phase. It may contain a few dead links and errors.
{: .text-center .notice--warning}

### Resolver name server
Resolver name servers are


### Authoritative name server
An Authoritative name server is responsible for providing definitive answers to DNS queries, meaning unlike resolver name servers it does not rely on cached responses from other name servers and instead will provide answers directly from it's domain zone configurations.

**For example:** within my Homelab my authoritative name servers are configured with DNS records for the home.olivando.me DNS zone.
{: .notice--info}