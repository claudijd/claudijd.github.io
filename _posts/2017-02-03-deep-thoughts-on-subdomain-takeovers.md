---
title: "'Deep Thoughts' on Subdomain Takeover Vulnerabilities"
layout: post
---

It's been about a year or more since I'd first heard about the term "subdomain takeover".  At the time, I was working through the triage of the first submissions of this type to our [bug bounty program](https://www.mozilla.org/en-US/security/web-bug-bounty/) and I came to the realization that this wasn't going to be the last I was going to see of this vulnerability class.

It's one of those types of vulnerabilities that is brought about by circumstances that inherently make it easy to fall into and its difficult to detect when you're vulnerable in any sort of predictable way because detecting cases is often vendor dependant.  Whenever you see these patterns with vulnerabilities, they tend to stick around for a long time and you can never really fully shake them because it's born from the natural chaos of how things work which gives them a much longer tail than your typical vulnerability.

In this post, I would like to reiterate how important and likely prolific this vulnerability is and will continue to be for some time.  I even went so far as referring to "subdomain takeover as the new XSS" when describing it to my bug bounty peers when we first started seeing these roll in.  Now, I could be overstating a bit and it will be hard to overcome the impact of XSS vulnerabilities on the Internet, but if you keep an eye on HackerOne bounties, you might also notice the recent uptick in subdomain takeovers as independant security researchers are catching on to this attack vector and realizing that subdomain providers and those that use them don't have a lot of good options right now.

**How does it work?**

The basic premise that makes this vulnerability possible is when you have a shared hosting provider that doesn't explicitly validate the subdomain (or VHOST) claiming process and an attacker claims your domain as their own without your approval.

Let's walk through a simple example to help flesh this all out.  Let's say you own the domain "example.com" and you want to host your blog ("blog.example.com") on a free hosting provider:

These are the steps you might follow to make this all happen...

1. Buy the domain (you might have already done this)
2. Setup your authoritative DNS servers (this might be setup for you, or you might have your own DNS servers)
3. Create an DNS record to point "blog.example.com" to free hosting provider server

After a little time to allow DNS to propagate and claiming that VHOST in the free hosting providers admin console, you can visit "http://blog.example.com" and see your blog in all it's glory.

If we take a look at this from an attacker perspective, you need to ask yourself, how did free hosting provider verify that the VHOST I provided is actually owned by me?  The answer is, they didn't.  This could mean that if someone was quicker than you in the claiming process they could beat you to the punch and claim your domain without your approval.  There are also cases where shared hosting providers allow the last person to claim the VHOST to assume control of the domain, in these cases you should ask your provider to fix this issue, or look for another provider to avoid chasing your tail forever.  Both of these cases on the surface can seem obsurd, but I can tell you with first hand experience they can and do happen.

**What shared hosting providers are affected by subdomain takeovers?**

Put simply, most shared hosting providers are affected by this type of vulnerability if they fail to put controls around VHOST claiming to ensure the claimer is authorized to claim (Examples: Heroku, Github Pages, SurveyGizmo, etc.).

If you are using a shared hosting provider for really any web services, there is a high chance that that provider is succeptible to this or at minimum doesn't do VHOST verifcation in a controlled way and leaves windows for this to exploited if mismanaged.  I will also say that if the shared hosting provider also allows you to install SSL certificates (especially wild card certs), this can really up the ante for potential impact scenarios.

**How can shared hosting providers prevent subdomain takeovers?**

Now, you might ask, is this even technical possible for them to do?  Of course it is, the best example that I can think of that closely demonstrates this pattern (and perhaps in more than one way) is that of [LetsEncrypt](https://letsencrypt.org/) and how they validate that the end user requesting a certificate is truly authoritative for that domain, which leverages the [Automatic Certificate Management Environment (ACME) protocol](https://tools.ietf.org/html/draft-ietf-acme-acme-04) to prove ownership.

If we look specifically at the ACME specification, it calls out a number of mechanisms to perform ownership/control validation...

- Put a CA-provided challenge at a specific place on the web server.
- Put a CA-provided challenge at a DNS location corresponding to the target domain.
- Receive CA challenge at a (hopefully) administrator-controlled e-mail address corresponding to the domain and then respond to it on the CA's web page.

If we look at these options, we might realize that the first one might be problematic because it relies on the web server already serving content, which might not work in VHOST claiming scenarios, but it would seem the second two are viable if we do some finding and replacing of the challenge provider and validator...

- Put a **shared-hosting-provider**-provided challenge at a DNS location corresponding to the target domain.
- Receive **shared-hosting-provider**-provided at a (hopefully) administrator-controlled e-mail address corresponding to the domain and then respond to it on the **shared-hosting-provider's** administration page.

**How can you better protect yourself from subdomain takeovers?**

Well, the first thing you can do is maintain a list of all the web services you host with shared hosting providers.  Depending on the complexity and breadth of your organization/setup this could be a rather simple task or a daunting one.

Once you have this list, you also need to maintain it and add/remove vendors from the list as services are created/transitioned/deprecated.  When doing so, you must be very sure to rigorously manage the DNS entries that point to these services because a dangling DNS entry pointed as a service provider where you're not claiming the domain is a recipe for getting tripped up on a subdomain takeover.

From an automation perspective, you might also be able to take these known list of providers and profile the behavior that each vendor demonstrates when the domain is unclaimed (usually a 403/404 error of some type, but not always) and then create an escalation internally to trigger some human action.  In cases where your service provider list is rather finite this is a good stategy, but in cases where your organization is decentralized, this can be a harder task to manage.  In any case, these actions are IMO a stop gap until shared hosting providers evolve enough to solve the claiming problem within their specific implementation.

**Other ways to skin this cat?**

I'll be the first one to say that I don't consider myself the source of truth and wisdom for anything.  I'm merely sharing my thoughts on this subject openly to help others understand and perhaps get other like-minded folks contributing to the discussion so we can better solve this problem for the community at large.  If you have ideas, email me and let me know what you think.  If it seems like a good strategy, I'd be happy to update this post to reflect that shared knowledge.
