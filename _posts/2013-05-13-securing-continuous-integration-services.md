---
title: "Securing Continuous Integration Services"
layout: post
---

Over the last couple weeks, I’ve had the distinct privilege to share some of my research surrounding continuous integration security.  The presentation was dubbed “Attacking Cloud Services w/ Source Code” and was presented at both SOURCE Boston 2013 and THOTCON 0x4, where I discussed a bunch of fun things like:

- Why I love Continuous Integration (CI) Services (especially hosted solutions)
- My perspectives as an open-source developer (some happy, some sad)
- What things could be possible if malicious code was fed to CI services
- A project I’m working on, called RottenApple, to help make things better

In this blog post I hope to capture some of the meat of the presentation for those who could not attend and use this opportunity to announce the first public release of RottenApple.

**Basic Terminology**

I guess the first big question (maybe for some) is what is continuous integration?

Continuous integration (to me) is a system or process that monitors source code repositories for changes.  If a change is detected (aka: a developer commits new code) the project is checked out built and tested to ensure that the software project still works.

Unit-tests (or "specs") are often used by continuous integration services to verify that everything still works in a software project. If something breaks, a unit-test will ideally fail and the continuous integration system will bubble that failure back up to a developer so that the problem can be addressed as soon as possible while the changes are fresh.

So, in short, developers use continuous integration to regularly checkout, build and run unit-tests on software projects for quality control.  Continuous integration is the entity performing the inspection and Unit-tests are what validate that each and every piece of software project works as expected.

**Why I love Continuous Integration**

Let me start by saying I haven’t always loved continuous integration.  In fact, at one point it was sort of a PITA.  When initially joining a group of Spiders that did a lot of development, I used to cringe at the idea of making a change that would cause the build to fail.  In the famous words of one of my co-workers, “this makes you a bad person”.

Anyways, beyond work stuff and fast-forwarding a bit, I was doing a little bit of open-source development stuff for fun and once I grew to appreciate the value of having continuous integration for my projects at work, I really wanted to apply the same techniques to introduce more quality control in my open-source “hobby” projects.

One of the challenging parts of doing development for fun is that you usually don’t have the same resources as you would if you were developing a commercial project.  This means you usually have a budget of $0 and you’re usually finding time to commit code between lunch breaks, while on the train or while pretending to watch TV.  Even though I knew continuous integration would be good for my projects, I didn't have the drive or the additional time to invest on it because it would mean less time writing code (what I really want to be doing).

**Travis-CI to the Rescue**

Thankfully, after a little while, someone introduced me to a service by the name of Travis-CI.  Travis-CI is a hosted continuous integration service for the open-source community.  It’s really easy to set up, mimics some of the stuff I have for my work projects and it’s **FREE**.  Now, there are other providers like this out there, but I have the most experience with Travis-CI, so I reference Travis primarily throughout.

In addition to being easy to setup and free, Travis-CI allows me to test my Ruby projects using a variety of different versions including 2.0, 1.9.3, jruby, rbx, ree, etc.  It also allows me to have a full build history for each change made to the project and the icing on the cake for me was that it would also build any submitted pull request and let me know if the proposed changes would break the project or not before I merge them into master.

**A Healthy Dose of Inspiration**

Around the same time that I was discovering all this fancy hosted-CI goodness like it was Christmas morning, I was also trolling the Aloha Ruby Conference videos and came across a video of a presentation called “Hacking with Gems” by Ben Smith.  The basic gist behind Ben’s talk was describing some of evil things you could do inside a Ruby gem and what sorts of things he tried to social engineer people, both actively and passively, to install his not so savory gems.

One of the most interesting components of Ben’s presentation was that he had a number of business cards made that simply stated “gem install aloha-ruby-conf” and placed them all around the conference.  It’s important to note that in this was a developer conference and it’s more than common place for people to install gems to try them out without really taking a close look at everything the gem does under the hood.  During the presentation, after describing all the evil things that someone could do with a simple gem install process, Ben provided a list of developer names that had installed his benign gem just to drive the point home.

Ben’s talk really got me thinking and eventually I came to wonder whether all these things (or similar techniques) were possible without having to rely on tricking a user or highly obfuscate my code.  This thought process led me back to continuous integration services.

**Attacking Continuous Integration Services**

When utilizing continuous integration services to build Ruby projects, it commonly boils down to executing a rake task either explicitly via “rake spec” or simply through running “rake” and having the default behavior to run the unit tests for a given project.  However, one of the things I noticed here was that my unit-tests themselves, even though I’m using rspec, which has some specific syntax for defining unit-test stuff, are just plain old ruby.  This got me to the idea, “what if I just added malicious code to my specs?”.   The system would most certainly execute my code and I’d get malicious code running on the CI.

 The first thing I decided to do before I got too far down the rabbit hole was to build my own CI server for testing purposes.  I did this because I really enjoy having a GitHub account and I do (as I mentioned above) love having a free continuous integration service building all my projects and I didn’t want to spoil that.  I didn’t want to piss anyone off and I didn’t want to feel bad when I did things that would be considered unethically hacking another organization.  I ended up building my own CI setup with Jenkins-CI, which is a widely popular open-source CI server used by a large number of organizations for building commercial software.

The setup was very basic where by I would push code to a private GitHub repository and my CI server would poll that project for any changes.  When I push malicious code up to my private GitHub repository, it would trigger a build on my CI server and then I would get that malicious code to execute on the CI.  It’s an extremely simple setup (in dev circles), but the nature of the situation means that the development of “exploits” for these environments, if you even want to call them that, is very easy.

Here are a couple example offensive-biased things that I talked about in the presentation:

- Breaking out of the build root (accessing neighboring project source code)
- Performing a port scan of the CI’s locally attached network (potential for pivoting behind the firewall)
- Authenticating back to GitHub using R-RW keys (potentially trojan the project)
- Popping a Reverse Shell (getting command-line access to the CI)

Although, throughout this description of all the offensive things you can do with Ruby projects (all of my examples) there is nothing to say that you couldn’t do these same techniques with building/testing any other language such as PHP, Python, Java, etc.  With such a fundamental level of trust here, the options are only limited by the trust-levels given to the CI to perform its activities.

**Introducing RottenApple**

![Rotten Apple](/images/rotten_apple.png)

After realizing that the nuances of CI security weren't going to be easily resolved for the masses (both hosted and self-hosted continuous integration services), I decided to start a project on GitHub called RottenApple with the hopes of at least moving us it into a better direction.  The main idea behind RottenApple is that you build the project on your continuous integration environment and it will test it and let you know where there are weaknesses.  Ironically enough, I’ve chosen to stick with a Unit-test concept where the roles are a bit reversed in that the unit-tests are actually testing the CI (not the code as they traditionally would).

After thinking long and hard about this tool, I decided to make it a bit more multi-purpose and implemented two separate, but related name-spaces in the RottenApple project; (1) RottenApple::Audit – for safely auditing a target CI environment and (2) RottenApple::Attack – for actively attacking a target CI environment.  I’m hoping that by having these two polar opposites that this project will help meet the needs of Developers, System Administrators, CI Providers (both hosted and internal installments) as well as Security Practitioners.

Below includes the current feature set of the project by name-space designation:

**RottenApple::Audit** has the following checks:

- Is the root user is being to build projects?
- Can malicious code steal your RubyGems API key?
- Could malicious code pivot to private networks?
- Can malicous code authenticate using your GitHub creds?
- Could malicious code receive instructions from a remote party or exfiltrate data from your CI?
- Can malicious code access other projects being built on the same server?
- Can malicious code steal SSH private keys?

**RottenApple::Attack** has the following features:

- Steal the RubyGems API key
- Flush IP Tables (aka: drop firewall rules)
- Install Software to aid in the attack process
- Make an unauthorized commit to master
- Perform an NMAP scan of a desired set to targets
- Throw/Shovel a reverse shell to get command-line access to the CI/CD
- Steal SSH private keys

I just recently published the source code for this project on GitHub, which can be found [here](https://github.com/claudijd/rotten_apple).  I hope that people check it out and find it useful.  Hopefully, with some good feedback and maybe a little help from the community I’ll be able to extend it to do more interesting things over time.

**Parting Thoughts**

Lastly, I just want to make myself as clear as possible that I don’t see the techniques that I’ve described above as “0-days”.  Continuous Integration services are open by design for a number of reasons, including making sure that they don’t inhibit the ability for developers to quickly have their code tested and validated for regressions.  However, it’s important to note that trust relationships do exist on these systems and can be abused if not carefully assessed.

I think we should do more to test continuous integration services for weaknesses to ensure the trust we have imparted them with is not abused.  I’m hoping that you checkout the RottenApple project, find it useful and send me a pull request.

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2013/05/securing-continuous-integration-services.html).