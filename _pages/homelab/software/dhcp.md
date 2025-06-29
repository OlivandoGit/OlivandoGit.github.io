---
permalink: /homelab/software/dhcp
title: DHCP

sidebar:
    nav: homelab-software-nav

classes: wide
---

**Please be advised:** This page is still in the draft phase. It may contain a few dead links and errors.
{: .text-center .notice--warning}


Dynamic Host Configuration Protocol (DHCP) is a networking protocol used to provide clients with network configurations.


Configurations may include: IP addresses (including lease information), subnets, default gateway address, DNS server address(es) and many, many more.

In most consumer deployments, DHCP servers are baked into the Internet Service Provider's (ISP) router and can be massively restrictive to use. This spurred me on to deploy my own DHCP server - one that would not be arbitrarily restricted in the most annoying ways.

