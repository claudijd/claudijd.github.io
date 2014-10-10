---
title: "Identify Crimeware Strains with Edit Distance"
layout: post
---

When trying to identify crimeware/malware, it's a good idea to design a multi-part system that deploys a variety of detection techniques to increase your chances of detection.  You can start with one technique and then layer on additional techniques as time and resources will allow.

In this short blog post, I’m going to share just one of those techniques (using edit-distance) that you can plug into your multi-part system to perform rudimentary detection for popular crimeware admin panel strains like Pony, Citadel, and Zeus.

**Edit Distance Basics**

Edit distance (aka: Levenshtein distance) is a term for determining how different two strings are from one an another. The basic idea is that we take String A (“bananas”) and String B (“apples”) and determine how many individual changes would be required to make the first string equal the second string. Each change can be an insertion, a deletion or a substitution.

For example, if we wanted to compute the edit distance between A and B we can do this manually like so:

1. Delete the ‘b’ (ananas)
1. Sub first ’n’ for ‘p’ (apanas)
1. Sub second ‘a’ for ‘p’ (appnas)
1. Sub second ’n’ for ‘l’ (applas)
1. Sub last ‘a’ for ‘e’ (apples)

So, assuming we took the most efficient path from bananas to apples, we have an edit distance of 5 between the two strings.

It’s a very simple concept, but how can something this simple help us identify crimeware?

Let’s start by getting our hands on some crimeware.

**Obtaining Crimeware Samples**

There is a metric ton of web-based crimeware that’s available in the wild, many of which we at Trustwave already classify using more sophisticated means.  I've taken 2 separate instances of 3 different “strains” of web-based crimeware (Pony, Citadel, and Zeus) from our malware repositories to demonstrate this technique.

These are the files I’m starting with:

- pony1
- pony2
- citadel1
- citadel2
- zeus1
- zeus2

Now that we have some samples, let's identify them with edit-distance.

**Identifying Crimeware Strains**

We start this process by identifying a baseline sample for each strain. Let’s use sample #1 for each strain.  We'll take the baseline examples and place them in a templates folder and then move the remaining items in a samples folder.  We can also add 100 normal HTTP responses and play a little game called "find the crimeware."

Now, on disk, our footprint looks like this:

- templates/
  - pony1
  - citadel1
  - zeus1
- samples/
  - pony2
  - citadel2
  - zeus2
  - random1..100

I've written this small proof of concept code to demonstrate the process with a couple performance and tuning tweaks added, including normalized edit distance and a sample qualifying pre-processor:

    require 'levenshtein-ffi'
     
    class String
      def size_percent(other)
        if self.size > other.size
          diff = self.size - other.size
          return diff.to_f / self.size.to_f
        elsif self.size < other.size
          diff = other.size - self.size
          return diff.to_f / other.size.to_f
        else
          return 0.0
        end
      end
     
      def close_enough?(other)
        return size_percent(other) < 0.1 ? true : false
      end
     
      def normalized_edit_distance(other)
        diff = Levenshtein.distance(self, other)
        if self.size > other.size
          return diff.to_f / self.size.to_f
        elsif self.size < other.size
          return diff.to_f / other.size.to_f
        else
          return diff.to_f / self.size.to_f
        end
      end
    end
     
    baseline_samples = {
      "pony" => File.read('templates/pony1'),
      "zeus" => File.read('templates/zeus1'),
      "citadel" => File.read('templates/citadel1')
    }
     
    Dir.glob("samples/*").each do |file|
      next unless File.file?(file)
      unknown = File.read(file)
      baseline_samples.each do |baseline_name, baseline_data|
        next unless unknown.close_enough?(baseline_data)
        if (percent_diff = unknown.normalized_edit_distance(baseline_data)) < 0.1
          puts "[+] #{file} is #{baseline_name} (#{(100 - (percent_diff * 100)).round(2).to_s}% match)"
        end
      end
    end

We can now use this script to quickly and efficiently identify the crimeware strains within the sample set in about 0.017 seconds:

    $ ruby test_samples.rb 
    [+] samples/citadel2 is citadel (99.46% match)
    [+] samples/pony2 is pony (99.2% match)
    [+] samples/zeus2 is zeus (99.62% match)

**Parting Thoughts**

Again, as I mentioned earlier in this post, this is a rudimentary technique for identifying web-based crimeware of this size and static content.  If an attacker wanted to deploy an evasion to such detection techniques, the effort involved would be trivial by adding additional content or simply obfuscating the content. In such scenarios, this is when having a more sophisticated algorithms and detection technology would be required for proper identification.

At any rate, at least for the time being it is possible to identify some web admin panels using an edit-distance technique. Maybe in the future we’ll see more crimeware authors invest futher in mechanisms for obfuscation in these admin panels as they do in other crimeware infrastructure components.

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2014/09/identify-crimeware-strains-using-edit-distance.html).