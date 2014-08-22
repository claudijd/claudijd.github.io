---
title: "Fingerprinting Ubuntu OS Versions using OpenSSH"
layout: post
---

Over the past couples weeks, I’ve been working on enhancing the operating system detection logic in the TrustKeeper Scan Engine. 

Having the capability to detect a target’s operating system can be very useful. Whether you’re performing a simple asset identification scan or doing an in depth review, this information helps you make more informed decisions.

In this blog post, I’ll be talking about a technique that that you can use to fingerprint a server operating system version using just its OpenSSH banner. Also, I’ll share some analysis I did using this technique and leveraging some publicly available scan data from the Critical.IO project that was released earlier this year.

**Anatomy of an Ubuntu OpenSSH Banner**

If you’ve ever been so curious to connect with netcat to an Ubuntu system that is running SSH, you would have seen something like this:

    ~ $ nc 192.168.101.139 22
    SSH-2.0-OpenSSH_5.9p1 Debian-5ubuntu1.1

Now lets take a look to at the individual components that make up the banner:

![Ubuntu banner](/images/ubuntu_banner.png)

Each element has a specific meaning and tells us more about the server:

- **Proto Version** – The SSH protocol version that should be used when connecting to this server.   These days, 2.0 is pretty standard.
- **OpenSSH Version** – This indicates what version of OpenSSH is installed on this server.  These are typically rev’d any time a new major operating system version is released to coincide with the most stable version of OpenSSH at the time.
- **Portable Version** – OpenSSH produces two variants; a “clean” version (minimal) and a “portable” version (more compatible).  The portable version indicates which revision of the portable release the server has installed.
- **SSH Comment** – This is not called out above, but is the equivalent to the combination of the build version and patch version shown above.   Basically anything after the space in the OpenSSH version is the optional SSH comment. This will vary significantly between operating system  type and packaging provider and is usually the most useful place for getting OS hints.
- **Build Version** – This is the build version that is specific to Debian ("deb")-style packaging.  This style of build version is seen in both Debian and Ubuntu operating systems.  It’s common to see a major operating system release use 1-3 of these build versions over it’s life span.
- **Patch Version** – This is the patch version of the deb package that is specific to only Ubuntu-based systems.  It describes the iterative package revisions involved in making small tweaks to the package and is commonly done to address bugs and security issues.  There could be many of these revisions within both major and minor releases of the operating system.

**Translate Banner Elements to Ubuntu Versions**

If you were to attempt to translate the above banner to a specific Ubuntu version at a glance, it would be very difficult because the versions we’re seeing in the banner do not appear to have any correlation to the Ubuntu version.  In the above example, my target is running Ubuntu 12.04, but none of the banner elements tell us this explicitly.

However, with some [very basic research](http://lmgtfy.com/?q=OpenSSH+5.9p1+Debian-5ubuntu1.1) we can find the following [Launchpad entry](https://launchpad.net/ubuntu/+source/openssh/1:5.9p1-5ubuntu1.1), which describes that this specific OpenSSH version, build version and patch version were built specifically for Ubuntu 12.04 ([Precise Pangolin](http://releases.ubuntu.com/precise/)).  These three elements effectively make a unique key that we can use to determine an Ubuntu version.

![Ubuntu Launchpad](/images/ubuntu_launchpad.png)

This technique was described in a [July 2010 blog post](http://auntitled.blogspot.com/2010/07/identified-ubuntu-version-from-ssh.html) by Worawit (sleepya), which includes examples that go beyond just Ubuntu.  Today, we’re going to talk about just Ubuntu, but it’s important to note that a similar process could be performed for other operating systems that provide consistent SSH comments in their package releases.

**Analyzing a big list of OpenSSH Banners**

The next step was to find a big list of OpenSSH banners and get started on translating SSH banners to Ubuntu versions.

Luckily, I came across HD Moore’s Critical.IO project, which had nearly a million SSH banners from Internet-wide scans performed back in March.  You can download the latest port 22 dataset from [this location](https://scans.io/study/sonar.cio).

The format of this data is in JSON, which makes it very easy to parse.  I used Ruby’s JSON library to parse each entry, extract its related banner and produce a histogram of which banners were the most common.  I excluded any banners that did not contain the string SSH, assuming they were not running a stock banner or they were not running SSH at all.

Of the 936,727 SSH banners in the resulting list, I identified 136,918 (15 percent) that appeared to be Ubuntu related.  Within that subset, there were 118 unique Ubuntu SSH banners.

I took these 118 unique banners and performed the translation technique above to produce mapping logic that can turn an Ubuntu SSH banner into its respective OS version.  Because the scan data that I used was from an Internet-wide scan, this gave me a strong sense that we’ve got really good sample of Ubuntu SSH banners to provide solid coverage the next time we need to translate an arbitrary banner to it’s respective Ubuntu version.

**Ubuntu Systems w/ SSH Running on the Internet**

Lastly, I took my mapping logic and put it to work so I could see what versions of Ubuntu were being used on the Internet during this time period.

![Ubuntu stats](/images/ubuntu_internet_stats.png)

One of the things I noticed about the translated data set was that 34 percent of all the fingerprinted versions were end-of-life versions of Ubuntu.

What’s important to remember about end-of-life software is that it no longer receives security updates, which is why updating software is an important tenant of PCI DSS.  The results of my analysis shows that a little over one third of all Ubuntu systems running publicly available SSH services are potentially vulnerable to any vulnerability released after the end-of-life date of their installed software.

If you’re ever curious whether an Ubuntu version is end of life, you can check the [Ubuntu releases page](https://wiki.ubuntu.com/Releases).

**Wrapping Up**

So as you can see this process is relatively easy to do, and it helps us understand more about a given target, which is always good.

You can perform this process ad-hoc on a penetration test and get a real sense for what operating system is installed on the target, which affects the subset of software that could be installed on it.  In particular, this process gives us clues to whether the operating system has available patches for specific vulnerabilities.

During the process of performing this research, I did reach out to the erratasec team, who [performed a more recent Internet scan](http://blog.erratasec.com/2013/09/we-scanned-internet-for-port-22.html) of SSH systems back in mid September.  I’m still awaiting their reply, but I hope to update this post with their data set if and when I can get my hands on it.

Additionally, I recently added this updated translation logic into the operating system fingerprinting code within the TrustKeeper scan engine.  We’re still tuning and tweaking a couple things, but I expect that these updates will go live within the next couple weeks.

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2013/10/fingerprinting-ubuntu-os-versions-using-openssh.html).