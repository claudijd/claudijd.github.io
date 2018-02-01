---
title: "Host header injections: what are they good for?"
layout: post
---

In the last couple weeks, I've noticed a increase in host header injection vulnerabilities being found/discussed and there a lot of questions swirling around "what can you do with them?".  I somewhat attribute this increase to more users finding Burp, doing a scan, and not fully understanding the impact of these vulns before they report them.  I've also seen [bug bounty programs award researchers](https://hackerone.com/reports/94637) for bounties for these vulns, which effectively chum the water and create a false sense of legitimacy.  In this short post, I wanted to quickly capture my current thoughts on this, as it's one of those things that looks bad on the surface (an attacker reflecting header content on your site, ZOMGWTFBBQ), but maybe it's not as bad as you think.

## What's a Host Header Injection? ##

I host header injection is exactly what it sounds like, and it's often seen demonstrated like this against say example.com...

REQUEST

```
GET / HTTP/1.1
Host: google.com <<=== Note that we're sending google.com to example.com
```

RESPONSE

```
HTTP/1.1 301 Moved Permanently
Content-Type: text/html
Location: https://google.com/
Content-Length: 178
Connection: Close
```

The basic outcome here is that example.com in this case is just saying "that's not me, you need to go look over here" and to the untrained eye, this feels like a clear [open redirect vulnerability](https://cwe.mitre.org/data/definitions/601.html).

## What's the impact? ##

Ok, so if we're trying to get better at this stuff, let's go through the thought excercise of how an attacker would actually exploit such a vulnerability to see if we can understand the impact to the site or it's users.

So, first things first, how does an attacker control host headers?

Well... I can think of a couple ways to set a host header:

1. Get them to click a link to [https://google.com/](https://google.com/)
2. Leverage a CRLF injection vulnerability to inject a header dynamically on add a host header in the response with the value of https://google.com/ (note this doesn't actually set the client's requested host header, but the host header in the response)
3. Perform a raw HTTP request and set the host header (note this is create for PoCs, but not real browser behavior)

Given those options (and I'm all ears for hearing more ideas on how to control a users host headers) those seem to be the only reasonable ways to set a host header.  When we look at these three, the outlook is rather bleak for an attacker.  If you can get a user to click a link, an open redirect is not needed.  If you have an CRLF injection you could just add a Location header and the host header injection is irrelevant.  If you can convince user to open a ncat/nc/curl and perform the request directly, that's great, but it has no impact on real browser behavior.

## Summary ##

In short, I think host header injections that result in an immediate 301 basically have no associated risk at all.  If you're an attacker, you are probably much better served by trying to convince a user to click your phishing link than to leverage a host header injection attack on a given user.  Again, I welcome feedback to be educated on this subject, but I wanted to write this down so I TLDR folks to my thoughts on the subject based on the information I have at hand.  I also started a [thread on twitter](https://twitter.com/claudijd/status/959074393241497602) about this to become more enlightened on the subject and will update this post if I come to any new realizations on this topic.

## Pedantic EOF/Concession/Add-On ##

I wanted to add one additional qualifying note that there can be cases where an application dynamically builds page content based on the content of the host header supplied by the client (these are often not the 301 scenarios I described above).  However, this behavior is generally considered a bad practice, and I would argue further that the inbound vector to exploit such a vulnerability is limited by the ability of an attacker to influence a users host header.  I will also concede that in some cases the application could be horrifically bad and present other risks, but in those cases I would argue that it's likely server-side risk and at that point the attacker could do whatever they want without needing to open redirect users (assuming they weren't chaining with other vuln classes).  If you're interested in reading more on the server-side risks, please checkout [@albinowax](https://twitter.com/albinowax)'s post on practical http host header attacks [here](http://www.skeletonscribe.net/2013/05/practical-http-host-header-attacks.html).

## UPDATE: but what about cache poisoning? (yikes, thanks internet/albinowax!) ##

One of the things I'd like to add some clarity on is the subject of cache poisoning.  It's an often overlooked space and it wasn't until I re-read [@albinowax](https://twitter.com/albinowax)'s post did I fully have an appreciation for how this would play out.  I will note that such a scenario requires some form of intermediate caching service and a lack of proper caching headers set that would instruct the CDN to cache those responses, but I feel strongly enough that this is the most logical vector of attack on a header injection via 301/302, that it deserves more lip service.

The attack would basically cut out trying to convince a client to request a malformed host header and repeat the host header injection attack from the attacker machine.  It would result in the same basic behavior as we described above (maybe a slight modification), but the attacker is depending on the CDN/equivalent to cache the response such that when a legitimate user makes a request to / in this case that they receive this poisened response...

```
HTTP/1.1 301 Moved Permanently
Content-Type: text/html
Location: https://google.com/
Content-Length: 178
Connection: Close
```

Here's a response from CloudFlare on this subject stating that they will cache 301/302's unless [they explicitly told not to cache the response](https://support.cloudflare.com/hc/en-us/articles/200168326-Are-301-and-302-redirects-cached-by-Cloudflare-).  So, to be clear, assuming the cache provider is willing to cache responses for requests with two entirely different host headers (I'm not sure how CloudFlare handles this or how other providers would handle this), then could be a viable vector of attack using just a host (or equivalent) header injection and a 301, but it's something that needs to be addressed via response cache headers and/or with the caching provider.
