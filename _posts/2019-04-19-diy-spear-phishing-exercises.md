---
title: "DIY Spear Phishing Exercises"
layout: post
---

Over the past couple years, I've had the great pleasure of slowly building out an internal red team capability at Mozilla.  It's been quite the adventure to effectively relearn how to do what I'd been doing for years as a consultant to working at an organization that puts so much of their passion into personal privacy and internal culture.

When I was a consultant, there was really no line I wouldn't cross to help demonstrate a risk for my client (at times, admittedly, we went to far).  That said, the amount of influence I had over each clients long-term strategies to affect risk were still only brief snapshots in time.  At Mozilla, it's been a much more immersive experience, that allows to be a member of a culture that I plan to exploit to demonstrate our own weaknesses.  I have this great opportunity to hone in on what myself and my amazing peers feel our our greatest risks and we get to dive head first into those challenges.

One of the things that we've identified as a risk to our organization (and really organization in 2019) is that of phishing.  Because of this, we've identified patterns and processes that allow us to mimic not only where we see current threats coming from, but also hypothesize what tomorrows threats will be and get started on simulating those attacks for our users to help them level up ahead of time.

In this post, I intend to share how you (at a high-level) could go about running your own exercises and share some of the lessons learned I've gained by conducting ours.  That said, I suppose a warning at this juncture is warranted that if you don't have authorization to conduct such tests, you should obtain authorization before conducting any spear phishing exercise.

## The Basics ##

When I think of phishing, I think of a few essentials that will be required to conduct an exercise.  These include:

1. Obtain approval (from whomever is authoritative for your organization)
2. A set of targets (people I want to phish that have something I want)
3. A means to communicate with those targets (typically email, but not always)
4. A phishing website (typically a clone of something the user is familiar with)
5. A motive for those targets to be phished (we're hoping proper training competes with this motive)

If you have all of the above, you could conduct a spear phishing exercise for your organization and do so with relatively zero/low cost.  I'd also note that having preliminary training to prepare your users for phishing attacks is useful, but not required if your first test is to simply establish a baseline understanding.

## Obtaining Approval ##

When we're talking about conducting your own phishing exercises, it's my own opinion that this is not a simple YES/NO exercise.  I've had industry peers who have obtained such YES/NO answers who went off to conduct an exercise and were fired/neutered(figuratively) when the organzation's expectations and the red team expectations were not aligned.  This is, in fact, an important process by which you want to seek out the key stakeholders within an organization and level-set to ensure they know why you would want to conduct a phishing exercise in the first place and what you want to learn.  This should include what assurances are in place to ensure a safe outcome and whether you're operating within the organizations cultural expectations (especially important if you are in-house).

In the early days of running these types exercises, it's best to self-govern a bit more and avoid the desire to do what I would call "hat tricks".  Basically, at the early stages, keep things simple, make reasonably tight constraints for your team to work within and you'll have a better chance of having a successful test and earning the trust of your stakeholders.  Earning this high degree of trust with your stakeholders will pay dividends when you are ready do perform "hat tricks" of your own.

## Target Selection ##

The goals you have set for your exercise above will likely dictate how you go about making a target selection.

Random selection is very helpful in the early stages of testing because it's easy to communicate that from N pool of users, you are making a truly random selection.  I've even found it helpful to publish the source of the scripts we're using for more technical stakeholders to (1) help build further trust and (2) squash any concerns about singling out a specific user.  Most organizations have a central identity store (Active Directory / LDAP / etc.), I'd recommend writing code to pull directly from these systems for the greatest amount of selection integrity and just making it easy on yourself.

As you work up to more focused exercises, the code you write to integrate with your organizations identity system can then help you leap frog into specific group targeting, where you can get more and more focused attack simulations.

When you are done, the goal is to have a specific set of users that you intend to phish to acheive your goal.

## Target Communication ##

Email has been and continues to be one of the most direct means of phishing users.  There are of course other means, such as ticketing systems and real-time communication platforms, but it's probably easiest to start with email.  Again, no "hat tricks" in the early stages.

We've utilized a couple different email sending solutions as a means to communicate with potential targets and get an enticing email into their inbox.  However, lately, the easiest so far has been using [Amazon SES](https://aws.amazon.com/ses/).  Here's the TLDR process for getting this piece working:

1. Buy a domain that fits your ploy
2. Go into Amazon SES and add a new domain (it will likely be sandboxed if this is a new account, requesting the limit be lifted takes a day or so, and is account-wide)
3. Go into your DNS provider settings and add the records requested from Amazon SES
4. Create template.json file, see the link above for more details
5. Create email.json file, see the link above for more details (best to use yourself as a target for development)
6. Upload your template, see link above for commands
7. Send your bulk emails, see link above for commands

Now, Amazon SES has a reputation dashboard, so it's important that when you are sending emails for your exercise that you are doing so to known good email addresses.  This should be pretty easy if you work at the company you are testing to get a good list to work with.  I personally prefer to send no more than 10 or emails in a batch for a few reasons; (1) I don't want to max out sending thresholds (2) If something goes wrong, it's less people impacted and (3) having too many targets can negatively affect your results - remember spear phishing implies a smaller target group.

## A Phishing Website ##

At this stage, you're only limited by your own creativity, but I'll continue to reminde you no "hat tricks" in the early stages.  This typically means a simple clone of a login page that maps to your ploy is a good first start.  You copy the pages HTML source and simply change it's login button capability to log the submission attempt to disk (or a remote datastore).

Some common rules for the road at this stage would be to ensure the phishing site is making use of transport encryption (Let's Encrypt certs are a good choice here), making sure you're not exposing a vulnerability that would allow a real attacker to hijack your test for malicious gain, and in my own personal opinion, it's a good idea to simply bitbucket those submitted credentials to /dev/null so you're not unnecessarily storing clear-text credentials of your users.

You may also want to see if your phishing website adopts strong web application security standards, so scanning your own phishing website with something like the [Mozilla Observatory](https://observatory.mozilla.org/) and making sure you get an "A+" is a famous past time of ours and helps us reduce the risk of the phishing site being abused.

## Target Motivation ##

Lastly, and perhaps most important, is providing motivation for your users to click the link and use your phishing simulation website.  This is always a weird paradox where on one level, during a training cycle, you want to ensure that your users are well equipped to deal with such scenarios.  But, on the other hand, in this particular stage, you're really trying to test that muscle that you hopefully trained earlier on to see how strong it really is.

In my experience, the best type of motivation is motivation that directly pertains to that users personal interests and immediate work function.  For example, something that might affect their paycheck or bottom-line in some way.  Those are the types of motivation that many users will blindly ignore and tend to mimic sophisticated threat actors hoping to exploit your users.

## Passing Thoughts ##

As seen above, doing your own phishing exercises are very attainable with minimal cost (We spent $0.01 USD in Amazon SES to run our last phishing exercise).  I'd even say you'll likely spend more time getting authorization, setting constraints, and brainstorming your ploy ideas than you will on the actual execution part.  It's a great way to evaluate whether your user training strategies are paying off and allows you to make minor (perhaps even major) tweaks to that strategy over time.

One last thing I'll do is go back to the "hat tricks" topic for second to make sure we're clear.  With this I'm suggesting that you don't make your exercises too complicated (ie. chaining multiple vulns to pull off something you can demo at DEFCON next year).  However, from an entirely different angle, I don't think you should make your too easy.  I often find that that is one of the more common responses we get from users who are operating on the perception that detecting phishing threats are easy ("just look for the misspeliings" or "the nigerian prince schemes").  If you're going to invest the effort to conduct a phishing exercise for your organization, it should match the type of threat actor that you are trying to defend against.  In most cases, these threat actors are capable of utilizing a spelling/grammar checker, have better ploys than to ask you to mail them cash, and often are doing a reasonable level of due diligence (such as buying like sounding domains and having legitimate HTTPS certificats on their phishing sites).

Good luck on conducting your own phishing exercises, please let me know if you found this high-level post helpful and please do pay-it-forward by sharing some of your lessons learned running them.
