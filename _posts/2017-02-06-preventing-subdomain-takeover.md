---
title: "Preventing Subdomain Takeovers for Shared Hosting Providers"
layout: post
---

Last Friday, I wrote a somewhat off the cuff post capturing my "deep thoughts" on [Subdomain Takeover Vulnerabilities]({{ base }}/2017/02/03/deep-thoughts-on-subdomain-takeovers/) and received an overwhelming amount of interest on the subject, which is exciting.  In that post, I eluded to some ideas about re-using patterns seen within the ACME protocol to perform subdomain ownership verification.

In this post, I'd like to outline in a bit more detail how a given shared hosting provider might seek to prevent this issue on their platform.  I'm going to keep it relatively generic and abstract from any specific provider implementation in the hope that this approach (or something similar) is adaptable enough to fit the needs of most service providers.

**What's the problem statement?**

Shared hosting providers don't have a mechanism to know whether **subscriber A** is authoritative for claiming VHOSTs on a specific domain.  This means that even though **subscriber B** is not authoritative for "blog.example.com" they may be able to exercise a first-to-claim, last-to-claim, or claim control bypass attack on that particular provider resulting in a subdomain takeover vulnerability.  This works because **subscriber A** already has their DNS for "blog.example.com" pointing to the shared hosting providers servers.  The impact here is that **subscriber B** now controls all the content for that particular site and can inflict confidentiality, integrity, and availability impacts on **subscriber A's** subdomain.  Examples can include but are not limited to hosting offensive content on the affected domain (porn, hate speach, etc.), eliciting secrets from the user (prompting for authentication), or subjecting the user to client-side vulnerabilities (a browser zero-day).

**What could shared hosting providers do to solve for this?**

A rather simple but effective mechanism could be to use DNS to determine who is authoritative to claim a given VHOST on a given shared hosting provider.  Let's work through this example as proving grounds...

Let's say that subscriber A has a UID of 1336 and subscriber B has a UID of 1337.

If **subscriber A** wanted to claim ownership for "blog.example.com" they would go into the shared hosting providers web portal and make an assertion that they wanted to claim that subdomain.  The response to this request would be an HMAC of the subdomain ("blog.example.com") and the subscriber ID ("1336").

**String:** "blog.example.com:1336"

**SecretKey:** "thisisasecret"

**HMAC(SHA512):** "aff4547a470031d..."

The shared hosting provider would then instruct subscriber A that they need to create a DNS [TXT record](https://en.wikipedia.org/wiki/TXT_record) for blog.example.com that returned text content of the above mentioned HMAC.  Once **subscriber A** adds said record, they can then complete the claiming process for that domain by verifying the DNS TXT record is present and correct.

If **subscriber B**, who is not authorized to claim "blog.example.com", attempts to claim this domain they will get to the point in the provisioning process where they are given the subscriber specific HMAC to implement in DNS.  **subscriber B's** attempt to claim this domain will be thwarted as **subscriber B** does not control DNS records for example.com.  This could address first-to-claim, last-to-claim, and some bypass scenarios I've seen in the wild with shared hosting providers.

**Caveats**

As with any simple solution, there are a set of limitations/challenges to overcome.  I'll try to list some of the ones I'm aware of to get us started (there are likely more I haven't considered yet)...

- Shared hosting providers would need to implement code that performs this verification step, which includes development time on their platform and behavior changes need to be communicated to their user base.
- Shared hosting providers would need to deal with pre-existing claims and have a time window in which an unverified claim becomes invalid if no action is taken to perform a verified claim.
- In the case of free shared hosting providers they would need to weigh the risk reward of reduced sign-up/conversion rates of subscribers using their platform with the added verification steps (instead of the instant success provided by an unverified claim process)
- In the case of a DNS response poisoning attack (via cache or MiTM), there could be potential for an attacker to use this process to claim a domain for which they are unauthorized for.

**Feedback?**

I've tried above to outline what I think are the problem, the solution, and the challenges.  If you have other ideas or things I have not yet considered that affect stakeholders in an unverified subdomain claiming model, please let me know.  I'd like to know what about this proposal works and more importantly what doesn't work so it can be ammended as a viable path forward for shared hosting providers.

**[UPDATE] 02/07/2017** - In discussing this issue with colleages more, we have determined the most common scenario for domain takeovers is when a VHOST becomes unclaimed (likely do to a partial decommission process) and leaves a dangling DNS entry intact.  Another pattern here for service providers attempting to prevent this type of takeover would be for the service provider to require that the DNS A record for "blog.example.com" be removed or no longer pointed at the service provider before the VHOST claim can be lifted.  This very simple procedure could work well for retroactive protection and virtually eliminate a whole class-type of takeover vulnerabilities with relatively minimal impact to end users.

**[UPDATE] 02/09/2017** - Over the past week, I've had the opportunity to discuss this strategy with a number of our service providers.  One rather interesting question that has come up is dealing with claim transition for subscribers, both within a service provider and from service provider A to provider B.  Let's walk through how that could be possible if you'd made use of both strategies above; (1) Require DNS TXT records for a successful claim and (2) Require DNS A record removal before a claim can be lifted.

**Subscriber Claim Transition (within a service provider)**

We have **subscriber A** on a given service provider and they want to transfer ownership to **subscriber B** on the same service provider.  **subcriber A** would initiate a transfer request to hand off ownership to **subcriber B** and during the transfer, they would be prompted with a new HMAC, as described above.  The DNS TXT record would then need to be transitioned to the new HMAC content that's based on **subscriber B's** subscriber ID.  Once the service provider can verify the HMAC content is accurate for **subcriber B's** HMAC **subscriber B** is prompted to accept the transfer.  If **subscriber B** accepts said transfer, then the claim transition will be complete.

**Subscriber Claim Transition (with different service providers)**

We have **subscriber A** on a service provider A and they want to transfer ownership to **subscriber B** on service provider B.  This actually is more simple that it would seem to indicate.  **subscriber B** would simply go through a typical claiming process (as described above) and would be challenged with a new HMAC on service provider B.  The DNS TXT record would then need to be transition to this new HMAC content, and upon completion service provider B will allow a successful claim.  The DNS A record is also transitioned to the new service provider and all traffic flows to them.  **subscriber A** then simply goes through and deletes the claim in service provider A (although they really don't have to, since the DNS is no longer pointing in their direction).

