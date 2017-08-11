---
title: "Quick and Dirty Unauthenticated Web Application Scanning"
layout: post
---

One thing I keep trying to repeat to myself over and over the past 6 months is a reminder to not let perfect be the enemy of good.  It's the engineer in me that wispers dissenting thoughts in my ear like, *"you know you can find more stuff if you were authenticated"* or *"you'd probably be better off integrating this into the development pipeline"*, but sometimes you just need something that you can point and shoot at an application to provide quick and timely feedback without saddling everyone around you with paralyzing perfection.

One of the items I've been focusing on lately is on a recipe for an MVP (Minimum Viable Product) to make DAST (Dynamic Application Security Testing) capabilities accessible for **ANYONE** who wants to make use of them.  This of course extends well beyond my work at Mozilla, but is still very relatable as we often have various web properties we need to assess, but sometimes the window for feedback is short and a stakeholder would much prefer getting initial feedback on an early PoC or experiment before it enters a more mature state to shake off some of that low hanging fruit.

In this post, I'm going to quickly demonstrate how you can perform an ultra quick web application scan using ZAP (a free and open source web application security scanner), and DVWA (damn vulnerable web application) to give us something interesting to look at.  This is just a general purpose technique for getting over the hump on doing some DAST-level scanning with the least amount of friction possible.

## Get your test application setup##

`docker pull infoslack/dvwa`

`docker run -p 80:80 infoslack/dvwa`

## Scan test your application with ZAP ##

`docker pull owasp/zap2docker-weekly`

`docker run -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-weekly zap-full-scan.py -r dvwa_report.html -d -t http://INSERT_YOUR_IP`

## View HTML Scan Report ##

Open `dvwa_report.html` in your favorite browser and you'll end up with a report with the results... 

<img src="/images/zap_report.png" style="width:700px;height:315px;"/>

I would like to re-iterate that this far from an end-all solution, I couldn't in good conscience tell you this is all you should do, but with the rate at which new technologies are being developed and the ease of being able to use open-source tools to improve our assurance of those web properties, I feel like this is a step in the right direction if the alternative is not to have any security testing of an application (again, not letting perfect be the enemy of good).

If anything, I hope you walk away with the understanding that getting some DAST coverage for your applications is possible with open source tooling and it's something that you can make use of right now with minimal time investment.