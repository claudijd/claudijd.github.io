---
title: "Anonymized Port Scanning w/ Docker+TOR"
layout: post
---

Admittedly, I've spent a large part of my career thus far doing security testing where stealthiness really hasn't been top priority and has taken a back seat to simply breaking in.

However, when I was out in San Francisco a couple weeks ago for the mid-year [Mozilla All-Hands](https://wiki.mozilla.org/All_Hands), I spent some time with members from the [TOR project](https://www.torproject.org/) who really got me thinking about how I could use [TOR](https://www.torproject.org/) to level up my game when it comes to doing certain classes of testing.  Again, it's not a need right now, but it's a curosity and an opportunity to learn something new.

Here were my requirements...

- I want the scanning environment to be ephemeral (spin up, scan, view results, throw away)
- I want to obscure my source IP address to reduce the likihood that I'll be blocked
- I want to be able to show others how to replicate this setup in a repeatable and safe way

**Solution**

Create a file named `Dockerfile` with the following contents...
{% highlight bash %}
FROM ubuntu:latest
MAINTAINER anonymous
RUN apt-get update && apt-get install -y tor nmap proxychains
ENTRYPOINT service tor start && bash
{% endhighlight %}

Build and Run the container in the same directory as your `Dockerfile` like this...

{% highlight bash %}
docker build -t anonymousnmap . && docker run -it anonymousnmap:latest
{% endhighlight %}

Perform your anonymized port scan like this...

{% highlight bash %}
root@08bd5fad6c8a:/# proxychains nmap -sT -PN -n -p 22 $(tor-resolve sshscan.rubidus.com)
ProxyChains-3.1 (http://proxychains.sf.net)

Starting Nmap 7.01 ( https://nmap.org ) at 2017-07-13 04:26 UTC
|S-chain|-<>-127.0.0.1:9050-<><>-45.55.176.164:22-<><>-OK
Nmap scan report for 45.55.176.164
Host is up (0.93s latency).
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 0.96 seconds
{% endhighlight %}

Exit the container with `exit` and it's like it never happened

I was hoping this would be a little more challenging or involved, but alas, it was actually quite straight forward.  What I like about this approach is that I can fire up a container just for this purpose and don't need to muck with my host settings or fire up a heavy VM for this task.

I also like that as soon as I'm done with my scan and type exit, all the state is magically erased from history without needing to take snapshots or restore things to a prestine state.  It's not only given me ideas how to better anonymize myself, but also how I can spin up quick and easy testing environments that are easier to replicate and throw away when I'm done with them.

**[UPDATE] 07/13/2017** - [Matt Kelly](https://twitter.com/breakersall/status/885498096213479424) had a good point on twitter that I should make a point as to why the '-sT -n -PN' were essential as part of this process.

Overall, I'd like to encourage people to make their own decisions on options for scanning, but I can see now why this is useful in the context of anonymized port scanning with TOR...

- **-sT (connect scans)** - this essentially means that our NMAP scanner is performing full TCP 3-way handshakes.  This is useful in the TOR space because we're going through a proxy here. -sS still works, but I wanted to be absolutely sure.
- **-n (don't resolve PTRs)** - there are two reasons for this (1) we don't want to make the scans any slower by hanging up on unnecessary DNS calls and (2) we don't want to take the risk of potentially having those DNS requests not go through TOR.
- **-PN (don't do ping scanning)** - there are two reasons for this (1) we don't want to make the scans  any slower  and (2) we don't want to miss hosts if they don't respond to ICMP.

If you're interested in other ways to optimize your NMAP scans through TOR, I would defer you to the excellent [NMAP docs](https://nmap.org/docs.html).
