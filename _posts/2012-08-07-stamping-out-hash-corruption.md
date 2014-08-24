---
title: "Stamping Out Hash Corruption, Like a Boss"
layout: post
---

Have you ever dumped [LM](http://en.wikipedia.org/wiki/LM_hash) and [NTLM](http://en.wikipedia.org/wiki/NTLM) password hashes from a Windows system using the registry and never been able to crack the hashes or [pass the hash](http://en.wikipedia.org/wiki/Pass_the_hash)?  If so, maybe this blog post will be of specific interest and/or importance to you.

A couple months ago, my good friend [Ryan Reynolds](https://twitter.com/reynoldsrb) of [Crowe Horwath](http://www.crowehorwath.com/) explained to me that often times he would extract password hashes from a Windows system via the Windows registry and the hashes would never crack.  He also discovered that when he pulled password hashes using other techniques, like LSASS injection, that he would get entirely different hashes, which in fact did crack successfully.

Here is just one example Ryan found when extracting hashes from [Metasploit](http://www.metasploit.com/) (via SAM/SYSTEM) and [Pwdump6](http://www.foofus.net/~fizzgig/pwdump/) (via LSASS injection).

- Hash Extraction via Registry using Metasploit
  - LM: 4500a2115ce8e23a99303f760ba6cc96
  - NTLM: 5c0bd165cea577e98fa92308f996cf45
- Hash Extraction via Injection using Pwdump6
  - LM: aad3b435b51404eeaad3b435b51404ee
  - NTLM: 5f1bec25dd42d41183d0f450bf9b1d6b

After hearing a bit about this problem and knowing full well that we were getting bad hashes, Ryan and I decided that we would put our heads together and see if we could solve this problem.

Here are some highlights that resulted from our research:

- The corruption problem has been around since the late 90’s
- This has affected Penetration Testers, Forensic Investigators and anyone else extracting password hashes from Windows Systems during this time.
- Nearly all the registry extraction tools we tested, including the following were affected:
  - [Metasploit](http://www.metasploit.com/), [Cain and Able](http://www.oxid.it/cain.html), Pwdump, [Fgdump](http://www.foofus.net/~fizzgig/fgdump/), [L0phtcrack](http://www.l0phtcrack.com/), Samdump, [Creddump](http://code.google.com/p/creddump/), Pwdump5, [Pwdump7](http://www.tarasco.org/security/pwdump_7/) and others.
- This behavior was observed on the following Windows Operating Systems:
  - Windows 2000, Windows XP, Windows 2003, Windows Vista, Windows 7, Windows 2008, Windows 8 Pre-Release and interim versions and service packs
- We discovered the source of this problem to be a logic flaw in how the registry information was being processed.
- We developed patches for tools that were open-source and are actively working with community developers to ensure that this problem is “Stamped Out”.

Here are the slides that we delivered at [DEFCON 20](https://www.defcon.org/html/defcon-20/dc-20-speakers.html#Reynolds) for your viewing pleasure:

<script async class="speakerdeck-embed" data-id="45e1ff703e2d01301b0d22000a8f8683" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

Additionally, for those who don’t like power point slides and would prefer reading RFC-like docs, we developed a 15-page white paper that describes the process in uber detail for your reading pleasure.

- [PDF Format](http://media.blackhat.com/bh-us-12/Briefings/Reynolds/BH_US_12_Reynods_Stamp_Out_Hash_WP.pdf)
- [Google Doc Format](https://docs.google.com/viewer?a=v&q=cache:rt8wxeHfLzQJ:media.blackhat.com/bh-us-12/Briefings/Reynolds/BH_US_12_Reynods_Stamp_Out_Hash_WP.pdf+&hl=en&gl=us&pid=bl&srcid=ADGEESgqsY4oS6avx8hkqNHmxoSB1-Ib8JMgAY2VAW6q3uWM1sMVIYMwl-tvxXNrlMQjQTphO6woA15eMQ6mCEkKP3N5cESe9lj-tDz2bI60N7BDbltWz2LXEWlkl5WE2IJGx5IkaSJH&sig=AHIEtbSWAmb93y2Q3owpkgpZREoOnewsCg&pli=1)

This post would not be complete if we didn't show you how this hash corruption stuff works.  Below we have provided the demonstration video that we used in our [DEFCON](https://www.defcon.org/html/defcon-20/dc-20-speakers.html#Reynolds) and [BlackHat](http://blackhat.com/html/bh-us-12/bh-us-12-briefings.html#Reynolds) presentations that shows corrupted hash extraction on Windows 8 using [Metasploit](http://www.metasploit.com/).

<iframe width="420" height="315" src="//www.youtube.com/embed/YoD8ebKEiKg" frameborder="0" allowfullscreen></iframe>

We hope that solving this problem for password cracking tools will help security professionals of all types obtain the correct password hashes from systems to evaluate their true state.

Here's a quick status of where we currently are with our patching efforts at the time of this writing:

- **Creddump** - Fixed in Creddump Creddump v0.3
- **Metasploit** - Fixed in Metasploit since pull request #587 (7/15/12)
- **L0phtcrack** - Fixed in L0phtCrack v6.0.16
- **Samdump2** - Fixed in Samdump v1.1.1 (fix pre-dates our research, but noting it anyways)
- **Fgdump** - Fix still in progress as SAM/SYSTEM variant of Fgdump is still BETA
- **Cain and Able** - Still awaiting response from Author
- **Pwdump7** - Still awaiting response from Author

If you have experienced this problem in the course of your work (or perhaps with a different tool not listed above) and the result of this research has directly helped you, we would really like to hear from you by commenting on this post.

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2012/08/stamping-out-hash-corruption.html).
