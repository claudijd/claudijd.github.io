---
title: "Signed Ruby Gems: A c7decrypt walk-through"
layout: post
---

As someone who’s responsible for a number of Ruby projects, both open-source and commercially developed, I’m always on the look out for new ways to improve how they are secured and delivered to end-users.

The most common method for delivering Ruby code to end-users is in the form of a Ruby gem.  A Gem is a simple container for code and other relevant bits that can be installed by end users with a single command. 

For example, if a user wants to install c7decrypt (a tool for decrypting Cisco passwords), they would install the gem like so:

    gem install c7decrypt

It has been my experience in working with security-focused Ruby developers that the topic of “signed gems” comes up every now and again, but I always fail to make time to check it out.  The reason why a developer would want to publish signed gems is because it helps ensure that the gem was not tampered with.

In this post, I'll explain how  I cryptographically signed the Ruby gems that are now produced by the c7decrypt project as of version 0.3.2.

**Create a Self-Signed Gem Cert**

First, you need to create the certificate you will use to sign your gems: 

    gem cert --build claudijd@yahoo.com
    Passphrase for your Private Key: 
    Please repeat the passphrase for your Private Key: 
    Certificate: /Users/jclaudius/.ssh/gem-public_cert.pem
    Private Key: /Users/jclaudius/.ssh/gem-private_key.pem
    Don't forget to move the key file to somewhere private!

Next, you need to set the permissions on those .pem files:

    chmod 600 gem-p*

**Add Public Cert to the Project**

Create a certs folder in your project and add the public cert to it

    mkdir certs
    cp ~/.ssh/gem-public_cert.pem certs/claudijd.pem

Add a cert reference to your projects gemspec file so it knows where to look

    s.cert_chain  = ['certs/claudijd.pem']
    s.signing_key = File.expand_path("~/.ssh/gem-private_key.pem") if $0 =~ /gem\z/

**Build the signed gem**

This is no different than building a gem normally, with the added step of having to supply the cert password, like so:

    gem build c7decrypt.gemspec
    Enter PEM pass phrase:
      Successfully built RubyGem
      Name: c7decrypt
      Version: 0.3.2
      File: c7decrypt-0.3.2.gem

Next, you can publish that gem to your gem hosting provider, like RubyGems:

    gem push c7decrypt-0.3.2.gem
    Pushing gem to https://rubygems.org...
    Successfully registered gem: c7decrypt (0.3.2)

**Install the signed gem**

Now that you have signed the gem and pushed it to the server, it’s time for end-users to do their part.  With users that don’t care about whether the gem is signed or not, their process doesn’t change:

    gem install c7decrypt
    Successfully installed c7decrypt-0.3.2
    Parsing documentation for c7decrypt-0.3.2
    Installing ri documentation for c7decrypt-0.3.2
    Done installing documentation for c7decrypt after 0 seconds
    1 gem installed

For users that care about whether the gem is signed, they will need to download the public cert file, trust it, and then install the gem with either a HighSecurity or MediumSecurity trust policy.

    gem cert --add <(curl -Ls https://raw.github.com/claudijd/c7decrypt/master/certs/claudijd.pem)
    Added '/CN=claudijd/DC=yahoo/DC=com'

    gem install c7decrypt -P HighSecurity
    Successfully installed c7decrypt-0.3.2
    Parsing documentation for c7decrypt-0.3.2
    Done installing documentation for c7decrypt after 0 seconds
    1 gem installed

In contrast, if a user who cared about the authenticity of the gem tried to install using the HighSecurity trust policy, and either we didn’t do this process or the gem was tampered with, they could expect this behavior, which would fail to install the gem:

    gem install c7decrypt -P HighSecurity
    Fetching: c7decrypt-0.3.2.gem (100%)
    ERROR:  While executing gem ... (Gem::Security::Exception)
        root cert /CN=claudijd/DC=yahoo/DC=com is not trusted

**Parting Thoughts**

So, as you can see, the process of building and using signed Ruby gems for your smaller Ruby projects is quite easy to do and it can help add some confidence for end-users who are more security conscious.  One thing I’m still looking into, and which may be a topic for a future post, is how to better make this process scale to my larger projects, such as those that have multiple gem dependency tiers.  If you have any suggestions on that topic, please feel free to contact me directly or submit a comment on this post.

**Reference:**

[http://guides.rubygems.org/security/](http://guides.rubygems.org/security/)

**UPDATE - 12/22/2014**

[@bascule](https://twitter.com/bascule) of [@Square](https://twitter.com/Square) made me aware of an effort they are working on to integrate the update framework (aka: TUF) with RubyGems for better Gem security.  I'm providing links to their excellent blog series and TUF if you'd like to learn more about what they're doing to improve gem security.

- [Securing RubyGems with TUF, Part 1](http://corner.squareup.com/2013/12/securing-rubygems-with-tuf-part-1.html)
- [Securing RubyGems with TUF, Part 2](http://corner.squareup.com/2013/12/securing-rubygems-with-tuf-part-2.html)
- [Securing RubyGems with TUF, Part 3](http://corner.squareup.com/2013/12/securing-rubygems-with-tuf-part-3.html)
- [The Update Framework (TUFF)](http://theupdateframework.com/)

Thanks [@bascule](https://twitter.com/bascule) and [@xshay](https://twitter.com/xshay)!

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2014/12/signed-ruby-gems-a-c7decrypt-walk-through.html).