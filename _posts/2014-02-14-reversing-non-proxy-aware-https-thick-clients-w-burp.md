---
title: "'Reversing' Non-Proxy Aware HTTPS Thick Clients w/ Burp"
layout: post
---

A little over a month ago, I published a Metasploit auxiliary module for brute-forcing Cisco ASDM logins.  Shortly afterwards, I received requests from a couple people to share how I was able to get access to the inside of the ASDM transport layer, which is encrypted with SSL.

Well, the short answer is that SSL isn’t really that much of a hurdle if the thick client you're reversing doesn’t verify the validity of the SSL certificates it’s being presented with.

The longer answer, and one I hope to answer during the course of this post, is that Burp makes “middling” non-proxy aware HTTPS thick clients (like ASDM) a pretty simple and straight forward process and I’ll show you how.

**Overview of the Process**

The process for “reversing” the transport communications of these thick clients is as follows:

1. Obtain a thick client that uses HTTPS transport
1. Obtain a copy of Burp
1. Start Burp with admin privileges
1. Add a Proxy Listener w/ Redirection and Invisible Proxying
1. Connect to proxy listener as if it was a server
1. Have fun…

Sounds pretty easy, right?  Well, let's get started then.

**Obtaining a Thick Client**

The first step to getting inside a thick client's transport layer is choosing a thick client to play around with. A "thick client", for the purposes of this post, is really any client application that you would download and run to connect to a server application.  I ended up using the Cisco ASDM client because I was focused on solving a specific problem, but the concepts and techniques can be repeated on most thick clients that don’t do certificate validation.  I encourage you to grab a copy of ASDM (if you’ve got an ASA) or pick a different thick client and follow along.

**Obtaining a Copy of Burp**

Go here and grab a free copy of Burp:

[http://portswigger.net/burp/download.html](http://portswigger.net/burp/download.html)

All the stuff we’ll be talking about that uses Burp can be accomplished using the free version.

**Starting Burp with Admin Privileges**

So with a bit of caution and your fingers and toes crossed, you’re going to need to start up Burp with admin privileges.  This may make some of you feel a little uncomfortable being that Burp is written in Java, but you’re going to need to give Burp additional privileges in a later step in this post.

If running Java with admin privileges is still tugging at your spidey senses, you are welcome to run without admin privileges and bind to a port in the ephemeral port range.  The only shortfall of doing this is that your thick client application may or may not allow you to change the server port that it connects to.  If it doesn't allow you to change the port, you will need admin privileges to proceed.

You can invoke Burp with admin privileges like this on Linux or Mac:

    sudo java -jar burpsuite_pro_v1.5.20.jar

You can also add your own creative flair to the options supplied by this command and give Burp more memory, but that is outside of the scope of what we’ll need to get the job done.

**Adding a Proxy Listener w/ Redirection and Transport Proxying**

Ok, now we’re getting to the interesting stuff.  This is going to require a little bit more explanation because it’s not a scenario you would see often.

In the default Proxy Listener configuration that most people who use Burp are familiar with, you are provided with a CONNECT proxy listener bound to the loopback adapter on port 8080.  This allows us to explicitly tell our browser where the proxy listener is and how to communicate with it.

The difference when using a thick client that is not proxy aware is the need for the ability to point the thick client to the proxy.  We need to do this because this is what will allow the proxy to see all the requests sent to the server.  We can do this by adding an additional proxy listener on the loopback adapter and configuring it to act as the legitimate server to which the thick client needs to connect.

Let's look at some screenshots on how to do this:

**Step 1**: Visit the Proxy => Options configuration tab

![Burp options](/images/burp_options.png)

**Note:** This is the default listener that is provided in Burp

**Step 2**: Click Add to add a proxy listener and add the bind port on the loopback

![Burp binding](/images/proxylistener_binding.png)

**Note:** You can use the server port (ie. port 443 in the case of Cisco ASDM) if you invoked Burp with admin privileges, if not, choose something in the ephemeral range and then use that port later when you connect with the client.

**Step 3**: Set a Redirect Host and Port and Enable Invisible Proxying

![Burp redirect](/images/proxylistener_redirect.png)

**Note:** The redirect “binds” your locally mapped port to the remote service forming an SSL proxy that Burp can inspect. The invisible proxying tells Burp that its interface should be an emulated web server interface rather than the typical proxy style interface that’s used by default.

**Connect to the New Proxy Listener**

Now that we have everything all setup and ready, we can now take the Cisco ASDM client and connect to the newly created proxy listener as if it was the Cisco ASA device.  So we fire up the Cisco ASDM client and point it at our proxy listener, like so:

![ASDM login](/images/asdm_login.png)

**Note:** Make sure you have the proxy intercept feature disabled, or authentication will timeout waiting for you to advance all the requests it needs to make to fully run.

As the ASDM client attempts to login, you will see that the site map within Burp will begin to populate with all the various paths that are used by the ASDM client to gather information and configure the Cisco ASA.

Here’s an example of what my site map looks like once ASDM has fully loaded.

![burp sitemap](/images/burp_site_map.png)

This tells us a lot about how the ASDM client works including its authentication process which in the very first request that is made to the firewall:

![burp post request](/images/burp_post_request.png)

As you can see the authentication is merely a post request that can be easily replicated in a script to quickly automate the process of credential checking Cisco ASA ASDM logins.  This is what was actually captured in the Metasploit module I mentioned above.

We can also leverage many of these paths that we can now see to perform the ASDM administration tasks directly without needing to deal with the restrictions imposed by the thick client interface.  This also makes us more aware of the server attack surface so we can perform a more thorough assessment of the application.

**Conclusions**

So there you have it. That’s how easy it is to “reverse” the HTTPS transport layer of a thick client using some built-in features within Burp Suite.  Not so bad, right?

Well, I encourage you to find a thick client that you use in your daily work and use this process to see how it works under the hood.  Maybe you can help automate some security checks for yourself or even use this process to help automate some administration tasks that were previously thought to be impossible.

Either way, I’d be really happy to hear from anyone who reads this post and follows this process to see what his or her thick clients are doing inside the transport layer.

See you next time!

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2014/02/reversing-non-proxy-aware-https-thick-clients-w-burp.html).

****Update as of February 24, 2014****

[@OrenHafif](https://twitter.com/orenhafif), one of my fellow researchers, reached out to me with some additional techniques that could be paired with the technique described in this post to make your experience even more enjoyable if you want to "hack harder."

I'll do my best to briefly summarize them here for the enjoyment of all...

1.) For thick clients that don't allow you to change the target server that use a FQDN to connect, you can modify your /etc/hosts file to point the FDQN of the server in question to the loopback IP.  This will cause the thick client to point to your proxy listener instead of the real server it would normally connect to.  This could also be accomplished with a local DNS server if you'd like.

2.) If you have a thick client that leverages the SSL certificate for server-side validation, but the initial request is sent over HTTP before being redirected to HTTPS, you can use Burp + SSLStrip to force the thick client to use an HTTP-only session and allow inspection of the transport layer.

Awesome stuff! Thank you [@OrenHafif](https://twitter.com/orenhafif)!


