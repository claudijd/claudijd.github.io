---
title: "Monkey Patching the Matrix"
layout: post
---

For those of you not familiar with monkey patching, it’s a mechanism to “extend or modify the run-time code of dynamic languages without altering the original source code”. Previously, I demonstrated how monkey patching can modify the logic of a Ruby gem dependency to work around hardcoded configuration settings. As you might imagine, this technique, if used in the movie the Matrix, would have been a great advantage to our favorite “hacker” Neo in his quest to be “The One”.

![Red Pill or Blue Pill](/images/matrix_hands.jpg)

In this blog post, I’ll be showing some real world Matrix code examples that demonstrate how you (a hacker) can upgrade your guns, learn karate, and avoid being shot by the lady in the red dress.

The choice is yours…

**Upgrading Your Guns (Method Override)**

![Neo Machine Gun](/images/neo_machine_gun.jpg)

If the Matrix were written in Ruby, the code to define a semi-automatic weapon could  look something like this:

    class Gun
      def trigger
        fire
      end

      private
      def fire
        puts "BANG!"
      end
    end

What you’ll notice about this gun implementation is that it honors the traditional constraints of a real semi-automatic weapon.   This means that every time the trigger is pulled the gun fires only one shot, like so:

    >> semiautomatic_gun = Gun.new()
    => #<Gun:0x00000102806a80>
    >> semiautomatic_gun.trigger
    BANG!

Now, if you were in a fight with, say, Agent Smith; you might need a slightly higher rate of fire. We can achieve this is by redefining Gun at run-time.   We do this by requiring in a monkey-patched version of Gun, which looks like the original, but with the trigger redefined to fire three shots instead of one:

    class Gun
      def trigger
        3.times {fire}
      end
    end

By requiring this modified code, we effectively convert our pre-existing semi-automatic gun to support 3-round bursts.

    >> require 'monkey_patched_gun'
    => true
    >> semiautomatic_gun.trigger
    BANG!
    BANG!
    BANG!
    >> semiautomatic_gun.to_s
    => "#<Gun:0x00000102806a80>"

You’ll notice that the object ID has not changed (same id), but magically the behavior is no longer the same.

Using this technique, we can define what methods do to alter pre-existing and new objects even if we don’t control the source code of gun.rb.  This ability allows us to change the run-time behavior of Gun and bend reality to our will.

**Learning Kung Fu (Instance Variable Stomping)**

![Morpheus Karate](/images/morpheus_karate.gif)

Now, I can attest as an avid movie watcher than many times you need more than big guns to beat the bad guy.  Sometimes you’re required to think on your feet and learn something new to overcome your adversary.  Luckily for Neo, Morpheous had a way to upload new skills so that anyone could learn things like flying a helicopter or learning Kung Fu.

If I had to imagine how Neo’s brain was originally implemented in the Matrix, it probably would have looked something like this:

    class Brain
      def initialize
        @skills = [
          "hide",
          "run",
        ]
      end
     
      def what_to_do?
        @skills[rand(2)]
      end
    end

Before Morpheus’ magic upload process, you could pretty much assume this would be a likely outcome for Neo if he ever ran into an agent.

    >> require 'brain'
    => true
    >> neo_brain = Brain.new()
    => #<Brain:0x0000010218aee0 @skills=["hide", "run"]>
    >> neo_brain.what_to_do?
    => "hide"
    >> neo_brain.what_to_do?
    => "run"

But... through the magic of what I can only assume to be monkey patching, Neo can actually learn new skills almost immediately.

    class Brain
      def get_new_skills
        @skills = [
          "round house kick",
          "karate chop",
        ]
      end
    end

    >> brain = Brain.new()
    => #<Brain:0x00000102802f98 @skills=["hide", "run"]>
    >> brain.what_to_do?
    => "run"
    >> require 'monkey_patched_brain'
    => true
    >> brain.what_to_do?
    => "run"
    >> brain.get_new_skills
    => ["round house kick", "karate chop"]
    >> brain.what_to_do?
    => "round house kick"
    >> brain.what_to_do?
    => "karate chop"
    >> brain.to_s
    => "#<Brain:0x00000102802f98>"

Again, you’ll notice our object ID’s are the same, but the brain now has new capabilities after getting new skills.

Using this technique, we can successfully overwrite instance variables that previously were not directly accessible interfaces of the brain.  This allows us to define new methods like we saw with the gun example, but it also shows how to override instance variables that determine how it ultimately behaves.

**Lady in the Red Dress (Inheritance Hijacking)**

![Lady in Red Dress](/images/lady_in_red_dress.jpg)

I’ve always wondered how agents in the Matrix can take control of other people. It reminds me of the simulation where  the woman in the red dress changes into an agent and points a gun right in Neo’s face.

Again, assuming the Matrix is written in Ruby, the woman in the red dress could be implemented like so:

    class Woman
      def do_something
        puts "Say Hello"
      end
    end

Now, we could always override the #do_something method using the same techniques we've already discussed.  This would be effective, but in the Matrix it would mean that agents would need to seek out each individual to make this an effective strategy for tracking down Neo.

However, a slightly more indirect solution is more likely.  Instead of going after the individual, lets go after the esssence of what makes them human.  In the Matrix this is some form of source code.  When we look at Ruby objects, they maintain an inheritance tree where many Ruby objects inherit from Object.  My guess is that in order for agents to take control over any person, they simply override primitive methods on Object like so:

    class Object
      def puts(data)
        super("Kill Neo")
      end
    end

The net effective of this is as follows:

    >> require 'woman'
    => true
    >> woman = Woman.new()
    => #<Woman:0x0000010119c268>
    >> woman.do_something
    Say Hello
    => nil
    >> require 'agent'
    => true
    >> woman.do_something
    Kill Neo
    => nil

Obviously, this is some really tricky business conducted by the agents.  I can only imagine why those that came before Neo weren't able to figure it out.  I guess it was only a matter of time before Neo realized that although Object is commonly thought of as the top of the food chain, it actually inherits from a more primitive beginning BasicObject.  BasicObject is is often overlooked and I'm guessing that's how Neo took control with this little gem once the agents finally tried to get inside him to see who this guy was:

    class BasicObject
      def id
       warn "game over"
       abort
      end
    end

Now, the key to this mystery is that when Neo finally ruffled Agent Smith's feathers, he had to figure who our hero really was, and that was his big mistake.  In Ruby, if you ever want to see what uniquely identifies an object you can always refer to its object id.  So when Agent Smith looked into who Neo really was he actually managed to set the rest of the matrix free, like so:

    >> require 'neo'
    >> neo = Neo.new()
    => #<Neo:0x00000103005858>
    >> neo.id
    game over
    $ 

**Parting Thoughts**

I've had a lot of fun talking about the Matrix and how monkey patching likely played an important part in the story line. I hope you enjoyed it too.

On a more serious note, I think monkey patching, especially in the Ruby community, can have a negative undertone. I'm sure you can find all sorts of blog posts telling you why it's bad.

My goal here is not to encourage developers to increase their use of monkey patching but to make security professionals aware of its existence.  As an application security assessor, monkey patching can be an extremely useful tool in digging further into an application than was previously thought possible.  Achieving this level of depth is done by "bending reality" just like we did today in the Matrix examples so that you can do more than you ever thought you could - just like Neo.

<iframe width="420" height="315" src="http://www.youtube.com/embed/aTL4qIIxg8A" frameborder="0" allowfullscreen></iframe>

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2014/08/monkey-patching-the-matrix.html).