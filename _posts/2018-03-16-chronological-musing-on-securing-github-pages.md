---
title: "Chronological Musing on Securing GitHub Pages"
layout: post
---

I'd like take a moment to review a few years of personal history around using GitHub Pages as a free hosting platform for web content and how that has evolved over time.  I'm also regularly on conference calls where I need to warn folks about being careful of "this" or "that" and maybe this will also serve as a good, "read this" article to help them learn from my experiences.

## The Beginning (Apr 3, 2013) ##

I guess I can start back to when I wanted to revive the blog again.  It was that eventual realization after a series of major life events (lots of speaking engagements, moving across country, getting married) and the following lull that made me realize I should blog more.

I have a lot to say, but my "writing about stuff" priority is always far lower than my "doing stuff" priority, so it's often an uphill battle to derive motivation to blog.  I've hosted wordpress blogs, tried writing my own blog platform, used blogger, and variety of other tools, but essentially I just wanted something where I could just write what I wanted to and make it public.  And that's how I found GitHub Pages (after seeing [@mastahyeti](https://github.com/mastahyeti)'s blog setup around that time) and then also [Jeykll](https://jekyllrb.com/).

The combination of both GitHub Pages and Jeykll were the perfect matchup for me.  I needed to do a little setup one evening and then I was off to the races with writing blog content in markdown.  GitHub Pages handled all of it and life was simple.  It was so easy I went through my prior SpiderLabs Research posts and ported those over to the new blog to prevent that work from disappearing someday.

At that time, the blog was accessible via this URL (http://claudijd.github.io).

## Custom Domains! (Apr 14, 2016) ##

Fast forward a few years, including a few more major life changes (sold a house, bought a house, changed employers) and I wanted to have a bit more polished blog identity. One of my first steps on that front was to give the blog a custom DNS name.

GitHub supported this type of setup and all you really needed to do was [add a CNAME file](https://help.github.com/articles/using-a-custom-domain-with-github-pages/) to your GitHub Pages repo which instructed their servers to serve my blog content on their servers under my custom domain name.  It was really convienent to have the hosting and code management system entirely free and in the same place.

After this point, the blog was accessible via this URL (https://blog.rubidus.com), as it is today.

## Enter Subdomain Takeovers (May 04, 2016) ##

Around this time I was really starting to get my sea legs at my new employer (Mozilla) and I was really enjoying one of my new roles being involved in the web bug bounty program.  In short, reporters find vulnerabilities in our websites, they report them to us, and we pay them for their efforts (more information [here](https://www.mozilla.org/en-US/security/web-bug-bounty/) if you're interested).

At this time, we received [one of our first reports of subdomain takeover](https://bugzilla.mozilla.org/show_bug.cgi?id=1267546) and to me it was just so simple and easy to exploit.  What made it even more interesting was that it was easy to end up in a vulnerable state because service providers (especially free ones like GitHub Pages) are more focused on user adoption of the service than adding security controls here could add unwanted friction in the user experience.

So yeah, there was sort of a rush on various shared hosting providers by bounty hunters to try and report all these issues. Sites hosted on GitHub Pages were a common occurance and much of the issues were tied more specifically to the lack of that CNAME file in either site turn up or turn down.  During that time, if you had a web property that had DNS pointing at a 3rd party hosting provider, but there was no VHOST claim on that provider, you were to ripe for the picking.

I gave talk about subdomain takeovers at OWASP Maine and wrote a couple blog posts ([first](https://blog.rubidus.com/2017/02/03/deep-thoughts-on-subdomain-takeovers/), [second](https://blog.rubidus.com/2017/02/06/preventing-subdomain-takeover/)) on this subject if you want to understand the subdomain takeover stuff a bit more.

## The Mozilla Observatory (August 25, 2016) ##

It had been nearly 4 months since my first [Mozilla Observatory](https://observatory.mozilla.org/) scan of the blog, which was met with lots of sadness.  I of course got an "F" out of the gate. I was serving web content using the custom domain of 'blog.rubidus.com' and it really only ever worked legitimately over HTTP because GitHub didn't support HTTPS for custom domains.

I wasn't happy with this situation, mainly because it made me a bit of a hypocrite asking people to secure their sites and use the Observatory as a means of doing so when my own blog was getting an "F" at the time.

## CloudFlare (August 26, 2016) ##

A day later, I stumbled on one of the massive never ending GitHub issue reports (maybe it was this [one](https://github.com/isaacs/github/issues/156)) that detailed that many users were feeling similar heartburn with the service.  I wanted HTTPS on my site, but I wasn't willing to self host because I'm cheap and lazy.

I believe it was in the above isse where I learned CloudFlare offered a free tier, which included free HTTPS cert management.  This of course was a compelling means of increasing my Observatory score and "level up" the blog's security.  Thankfully, it was a pretty quick process, I just switched over my DNS hosting from the registrar I was using to CloudFlare and followed a few UI wizards.  The basic idea here is that CloudFlare acts as a reverse proxy for your website content.

Client === HTTPS ==> CloudFlare ==> HTTPS ==> GitHub Pages

And what you would often do here is point DNS at CloudFlare, and they would do the backend work to reach out to GitHub and host your site content under the new domain.  That of course worked great, but at the time I didn't realize this setup with a CNAME file in the repo was actually a security issue (I learned this much later).  However, if you look at the setup a bit closer you'll see what I mean.

1. Client does DNS lookup for blog.rubidus.com and get CloudFlare IP
2. Client makes HTTPS request for CloudFlare
3. CloudFlare makes HTTPS request to claudijd.github.io
4. GitHub Pages 302's to http://blog.rubidus.com (whoopsies!!!)
5. Client makes HTTP request ....

So, you end up with an HTTPS chain with an HTTP hop right in the middle, which makes your site vulnerable to man in the middle attacks at that hop.  As I noted above, I never realized this and ran the blog in this state for a little over a year, while turning on HTTPS redirect and HSTS requirements along the way.

## CloudFront + Lambda (November 9, 2017) ##

At this time I had been running the site on CloudFlare using a free acount and I was pretty happy with the setup, short of the periodic reminder that I was still a hypocrite asking people to get A's using the Observatory but rolling with a measly "D" on my personal site until one of my peers at work ([kang](https://twitter.com/kangsterizer)) made me aware of a way to really level up my Observatory score using a blend of Amazon CloudFront and Viewer Response Lambda functions ([reference architecture example here](https://github.com/mozilla/infosec.mozilla.org/blob/master/aws/cloudformation.yml)).

One of the really interesting outcomes of this effort (amusing for me) was that part of the solution involves removing the GitHub CNAME claim in the repo, which conincidentally gave me the willies because of it's historical subdomain takeover relevance, but because the CloudFront strategy includes using https://claudijd.github.io origin and does not preserve the original host header from the request, this means that by leaving the CNAME intact you'd end up with a 302 loop.

The setup is pretty easy, and works very similar to the CloudFlare setup, but with added control over response.  We also quickly ported a number of sites over to use this setup at work.  For personal use, it's great because I don't have to worry about managing a server/services and my overall CF+Lambda spend is < $1 a month (far cheaper than self-hosting options).  And another benefit is that I don't have to be a hypocrite anymore.

## RubyGems Blog (March 2, 2018) ##

In my spare time, I've also been helping the RubyGems/Bundler team to operate their [Bug Bounty Program on HackerOne](https://hackerone.com/rubygems).  Part of that is also taking a closer look at security stuff in general and one of things I really wanted to do was add HTTPS to the RubyGems blog.  Thankfully, [indirect](https://twitter.com/indirect) and [drradcliff](https://twitter.com/dwradcliffe) beat me to the bunch a few weeks ago and got it all setup with Fastly (those guys are amazing!).

However, one of the things I noticed in this process was that we had not lifted the CNAME claim in the repo and this tickled my spider sense from recently having to lift the CNAME claim on a number of sites in a November push.  And sure enough, that same issue reared it's ugly head, and we ended up with an HTTP hop in an HTTPS chain (note: Fastly by default preserves the original request host header, which is slightly different behavior than CloudFront).  I filed an issue [here](https://github.com/rubygems/rubygems.github.io/pull/32), but I think we're going to be moving to a new strategy there soon, so we're going to leave it be for now.

## What's Next (?) ##

I've heard some talk about GitHub moving to support HTTPS for custom domains sometime in the future (at least in a beta capacity).  That would make for a compelling argument for someone seeking to HTTPS their custom domain GitHub Pages site, which seems to be the main concern for users of the service.  However, I suspect at least in the early iterations of this, you will likely not have full control over HTTP headers, which at least for my blog, is a requirement.

I'm also hopeful that in the future free hosting providers offer HTTPS certs by default and also give options to their users to select appropriate security headers to match their security needs.  I'm also excited to see where this saga leads next for securing GitHub Pages and how that will impact how I manage the security of my personal blog and other web properties in the future.