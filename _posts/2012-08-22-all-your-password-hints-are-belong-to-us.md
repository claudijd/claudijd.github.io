---
title: "All Your Password Hints Are Belong to Us"
layout: post
---

This past weekend I ended up coming into the SpiderLabs office and “nerded out” with my good friend [Ryan Reynolds](https://twitter.com/reynoldsrb) to follow-up on the [research we released at DEFCON and BlackHat](/2012/08/07/stamping-out-hash-corruption/) this year.  As some of you may already know, our research was focused on corruption of LM and NTLM password hashes when they were extracted from the Windows registry (specifically, the SAM) by many tools.

In the process of going through one our test cases on a Windows 8 machine on Saturday, we stumbled across a new key (“UserPasswordHint”) that we had not seen before in the SAM database.  To both of us, this seemed like a rather odd, but intriguing key that would be cool to explore a bit more “at some point”.

Well… being the impatient weirdo that I am, I could not wait and dove in headfirst as soon as I got home that night.  This blog post will cover some of the details on how to extract/decode User Password Hints from the Windows registry and how I extended Metasploit’s Hashdump tools to incorporate this information.

**Inside the Registry Data**

For starters, the “UserPasswordHint” key on Windows 8 (and Windows 7) is stored in the following location:

HKLM\SAM\SAM\Domains\Account\Users\<userkey>\UserPasswordHint 

If you’re running with SYSTEM access (you can use your imagination here), you can read this key easily by doing a reg query like so:

![Password Hint Registry](/images/password_hint_registry.jpg)

In first looking at the storage location here, I was a little disappointed thinking that the hint was encrypted in some way until I noticed the pattern of zeros.  Having dealt with a fair amount of PHP malware in the last couple months, one of things the “baddies” do is chunk up their payload data into individual characters and then encode them in their ASCII numerical representation.  Well in looking at this registry value, it seemed to follow a similar approach, so I wrote a little decoder in Ruby to see if I could learn this users password hint.

Here is the decode method I whipped up to do this:

![Password Hint Decode](/images/password_hint_decode.png)

Now you can use the value stored for this user and translate it back to the clear-text hint, like so:

![Hint Decode irb](/images/hint_decode_irb.png)

Although this stuff looked a bit unreadable on the surface we can now see that it can clearly be decoded and could be used by tools that extract information from the SAM.  This seems like it would be very helpful for penetration testers by giving them more insight into what the user’s password might be, so I decided to take it one step further.

**Extending Metasploit's Hashdump Tools**

In our recent hash corruption research, we ended up making a number of changes to the Metasploit Hashdump tools (hashdump.rb and smart_hashdump.rb) and became very familiar with how they work.   As such, it just made sense to try integrating user password hint decoding with these tools so I sent a [GitHub pull request](https://github.com/rapid7/metasploit-framework/pull/706) on Sunday night to the Metasploit team to add this functionality. 

Here is a preview of what these new additions will give users when dump hashes from a Meterpreter session:

![Metasploit Password Hint](/images/metasploit_password_hint.jpg)

As of yesterday, these password hint extraction features were merged into the Metasploit Project.  I would like to extend a special thanks to [@_sinn3r](https://twitter.com/_sinn3r) and [@TheLightCosine](https://twitter.com/TheLightCosine) for their ideas, de-turding and review of my code.  Also, props to [@_sinn3r](https://twitter.com/_sinn3r) for pointing out that Window XP also stores user password hints in another location in the registry in the decoded form (this was also included in the recent Metasploit improvements).

**Parting Thoughts**

I had a lot of fun with this little detour through the Windows Registry for a couple hours.  If you use these tools and these new fangled hints help you out in some way, I would love to hear your story.

That is all for now.  I hope to have more Windows Registry hijinks and other “hax” to talk about here in the near future, so please stay tuned.

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2012/08/all-your-password-hints-are-belong-to-us.html).
