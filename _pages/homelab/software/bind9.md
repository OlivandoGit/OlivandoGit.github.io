---
permalink: /homelab/software/bind9
title: Bind9

sidebar:
    nav: homelab-software-nav

classes: wide
---


Bind9 is an open-source project maintained by the Internet Systems Consortium (ISC), which has evolved to be a very flexible, fully-featured DNS system. It boasts of being one of the most widely deployed DNS solutions, as well as being one of the oldest solutions available.

Bind9 is completely configurable via JSON files - with the main configuration file called ```named.conf``` stored at ```/etc/bind``` by default

In my Homelab, I am using bind9 as both an authoritative name server and a resolver (recursive) name server.

**Warning:** My deployment of Bind9 makes use of containerisation via Docker and Docker-compose. For further reading on these topics, please consider reading [Containerisation](/homelab/software/containerisation), [Docker](/homelab/software/docker) and [Docker-compose](/homelab/software/docker-compose) from the sidebar. Thank you
{: .notice--danger}

**Github:** Incase you didn't know, my homelab is open-source and can be found on my [Github](https://github.com/OlivandoGit){:target="_blank"}.
This projects files can be found at [Github/Homelab/Services/Bind9](https://github.com/OlivandoGit/Homelab/tree/master/services/bind9){:target="_blank"}
{: .notice--warning}

## Resolver DNS server
My Homelab DNS resolvers follow the "Selective Forwarding Resolver Configuration" model defined on the [Bind9 documentation pages](https://bind9.readthedocs.io){:target="_blank"}. This means that the server will look into each DNS query before deciding to either forward it to one of my authoritative name servers or answer the DNS query using recursive DNS resolution.

**For Example:** Any DNS queries for home.olivando.me will be forwarded to one of my authoritative name servers, but any other DNS queries (ie general internet traffic) will be solved recursively.
{: .notice--info}

My [Homelab topology](/homelab/topology) calls for the deployment of two resolver DNS servers so that I have fault tolerance of DNS within my home network. There is no syncing set up between the two resolver servers' DNS caches as (in my use-case) the DNS protocol is more than performant enough to outweigh any additional complexities that would come with syncing the caches. This also enables me to configure both of the resolver DNS servers with just one config file, which cuts down on maintenance.

##### Resolver DNS server configuration file - named.conf
First let's have a look at the main configuration file for my Resolver DNS servers

``` json
acl internal {
    192.168.1.0/24;
};

options {
    directory "/var/bind";

    version "not currently available";

    allow-query { internal; };

    forwarders {
        8.8.8.8;
        9.9.9.9;
    };
};

zone "home.olivando.me" {
    type forward;

    forwarders {
        192.168.1.253 port 2053;
        192.168.1.252 port 2053;
    };

    forward only;
};
```

The first part of this JSON defines an access control list with the name "internal" which is used later in the configuration in order to limit DNS queries.

The next part of the JSON defines the default options for the DNS server.
- ```Directory```: Defines the base directory for the files used by the server.
- ```Version```: This is set to unavailable as a best practice to mitigate version based vulnerability attacks.
- ```Allow-query```: This is used along with the access control list defined earlier to limit the server to only answer DNS queries from my local ip range.
- ```Forwarders```: The ip addresses of the next servers in the DNS recursive lookup chain.

The final part of the JSON uses a zone configuration to overwrite some of the default values defined above. This is to allow my resolver DNS servers to forward queries to my Authoritative DNS servers
- ```type forward```: Sets the type of this DNS server as a forwarder within this zone.
- ```forwarders```: The ip addresses of my authoritative DNS servers. As they use a non-standard port for DNS queries this is also defined (explained in the Authoritative DNS server section).
- ```forward only```: Tells the server to only forward any queries within this zone.

And thats all it takes to configure a (basic) Bind9 selective forwarding resolver, there are many more options that can be added to this configuration file to fit any number of different use cases. Additional options can be found on the [Bind9 documentation pages](https://bind9.readthedocs.io){:target="_blank"}

##### Resolver DNS server docker-compose file
Next let's have a look at the docker-compose file for my Resolver DNS servers

``` yml
services:
  bind9-resolver:
    container_name: bind9-resolver
    image: ubuntu/bind9

    environment:
      - BIND9_USER=root
      - TZ=Europe/London

    network_mode: host

    volumes:
      - /etc/bind/resolver:/etc/bind:rw
      - /var/bind/resolver:/var/bind

    restart: unless-stopped
```

This docker-compose file defines a single service named bind9-resolver and sets the container name to also be bind9-resolver.

The image I have chosen for this container is actually maintained by Canonical (The maintainer of Ubuntu) rather than ISC as the image maintained by ISC is maintained on a best-effort basis. In my mind, this means that any long-term versions of this image will potentially be better maintained and less likely to contain unpatched vulnerabilities.

When using the ubuntu version of the bind9 image, there are a couple of environment variables that should be defined: 
- ```BIND9_USER```: Define the user to start the bind9 application
- ```TZ```: Timezone selection

The network mode is set to host to allow the resolver DNS server to also serve ipv6 DNS queries as ipv6 is not currently officially supported by docker.

Volumes are set to allow the container to see the config file that is stored on the server as well as to allow easy recovery of any files created by the server.

Finally the restart policy is set.


##### Resolver DNS server deployment.
All thats left to do is to move the ```named.conf``` file into ```/etc/bind/resolver``` on the server and then use the docker compose file to create this service on the remote host.

For this, I created a [convenience script in bash](https://github.com/OlivandoGit/Homelab/blob/master/bind9/resolver/setup.sh){:target="_blank"} which was especially helpful during testing on multiple servers.


## Authoritative DNS server
My Authoritative DNS servers are separated from my resolver DNS servers as a best practice as defined by ISC. They do technically call for entirely separate hosts for this separation, however as there will never be any external access to my DNS servers, I felt comfortable in having them as separate containers on the same hosts.

This does cause some issues with port mapping, as both the authoritative DNS server and resolver DNS server will be trying to listen on port 53 by default, but we can easily use Docker's port forwarding to resolve this issue.

**For Example:** As defined in the resolver DNS server configuration file, I will be using external port 2053 to map to port 53 my authoritative DNS servers.
{: .notice--info}

Similar to with Resolver DNS servers, my [Homelab topology](/homelab/topology) again calls for the deployment of two authoritative DNS servers so that I have fault tolerance of DNS within my home network. Syncing between these servers can be easily accomplished by configuring one server as the primary DNS server, and the other as the secondary DNS server. This will require two separate configuration files.

##### Authoritative DNS server configuration files - named.conf
First let's have a look at the main configuration file for my Primary Authoritative DNS servers:

``` json
acl internal {
    192.168.1.0/24;
    
    172.16.0.0/12;
};

options {
  directory "/var/bind";

  version "not currently available";

  allow-query { internal; };

  allow-query-cache { none; };

  recursion no;
};

zone "home.olivando.me" {
  type primary;

  file "home.olivando.me.zone";

  allow-transfer {
    192.168.1.252;
  };
};
```

The first part of this JSON again defines an access control list which is also used later to limit DNS queries.

The first three global options are the same as discussed above in the resolver DNS servers' config, but the final two are new:
- ```allow-query-cache```: Disables the caching aspect of this server as it is not needed on an authoritative server
- ```recursion no```: Prevents the server from performing any recursive DNS lookups as this will be handled by the resolver DNS servers.

The final part of the JSON configuration is used to define the DNS zone for which this DNS server is authoritative.
- ```type primary```: Sets this server as the primary DNS server for the zone.
- ```file```: Provides the location of the file that contains the DNS records for this zone - this will be discussed later.
- ```allow-transfer```: Defines the secondary servers for this DNS zone.

For the sake of completion, here is the configuration file for my secondary DNS server:
``` json
acl internal {
    192.168.1.0/24;

    172.16.0.0/12;
};

options {
  directory "/var/bind";

  version "not currently available";

  allow-query { internal; };

  allow-query-cache { none; };

  recursion no;
};

zone "home.olivando.me" {
  type secondary;

  file "home.olivando.me.saved";

  primaries {
    192.168.1.253;
  };
};
```
This should all be self-explanatory now, and if not then please refer to the explanations below the primary DNS server configuration file

##### Zone records file
Next let's have a look at the zone configuration file for my authoritative DNS servers:

```
$TTL 2d

$ORIGIN home.olivando.me.

@	            IN	    SOA     home.olivando.me. ns.olivando.me (
                                                    2024052601; serial
                                                    12h ; refresh
                                                    15m ; retry
                                                    3w ; expire
                                                    2h ; minimum TTL
                                                    )

	        IN      NS      ns1.home.olivando.me.
                IN      NS      ns2.home.olivando.me.

ns1             IN      A       192.168.1.253
ns2             IN      A       192.168.1.252

```
Here TTL defines the length of time that these resource records should be cached by resolver servers. I have set this to two days as I do not intend for records to be reused often, so am not worried about any caching issues.

Next the origin is defined. This is the sub-domain of the zone, and will be the final part of any of the records defined here.

The first resource record in this file is denoted by the @ symbol which is used to define the origin of the domain. This can be used to set up key characteristics of the zone. These characteristics are later used by the secondary DNS server in order to decide when to sync the resource records of the servers.
- ```serial```: This serial number is compared by the secondary to decide whether to sync. If the serial of the primary is higher than that of the secondary, then a transfer is completed.
- ```refresh```: Defines how often the secondary should poll the primary.
- ```retry```: Sets the retry timer if the secondary cannot contact the primary.
- ```expire```: Indicates when data is no longer authoritative. The secondary DNS server will stop responding to queries if it does not receive an update from the primary.
- ```minimum TTL```: Again used by the secondary DNS server to decide when to initiate a zone transfer

The next pair of resource records define the name servers for this zone as a fully-qualified domain name (FQDN).

The final pair of resource records provide the A (ipv4) records for the two name-severs as defined by the FQDN above.

##### Resolver DNS server docker-compose file
Finally let's have a look at the docker-compose file for my Resolver DNS servers:

``` yml
services:
  bind9-authoritative:
    container_name: bind9-authoritative
    image: ubuntu/bind9

    environment:
      - BIND9_USER=root
      - TZ=Europe/London

    ports:
      - "2053:53/tcp"
      - "2053:53/udp"

    volumes:
      - /etc/bind/authoritative:/etc/bind:rw
      - /var/bind/authoritative:/var/bind

    restart: unless-stopped
```
This file is very similar to the file used with the resolver DNS servers. The only difference here is that Docker port-forwarding is used to port-forward the host's port 2053 to the container's port 53. This was to resolve the port overlap issue discussed earlier.

##### Authoritative DNS server deployment.
All thats left to do is to move the correct  ```named.conf``` file into ```/etc/bind/authoritative``` on each server, move the zone file to ```/var/bind/authoritative``` on the primary dns server, and then use the docker compose file to create this service on the remote hosts.

For this, I created another [convenience script in bash](https://github.com/OlivandoGit/Homelab/blob/master/bind9/authoritative/setup.sh){:target="_blank"} which was even more helpful this time due to the need for multiple configuration files.
