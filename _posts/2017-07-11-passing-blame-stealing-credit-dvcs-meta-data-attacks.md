---
title: "Passing Blame & Stealing Credit w/ DVCS Metadata Attacks"
layout: post
---

I got into a lively discussion with the [OWASP Maine](https://twitter.com/OWASPMaine/) crew tonight about meta-data attacks against distributed version control systems ([DVCS](https://en.wikipedia.org/wiki/Distributed_version_control)) and it simply sparked an interest in writing this hopefully short little post to highlight how it can be abused, so here goes...

**The Basic Premise**

DVCSs like [Git](https://git-scm.com/) and [Mercural](https://www.mercurial-scm.org/) are as the name suggests a distributed version control system.  This means that when I perform a checkout of a given project, I have a fully portable copy of that repository.  Once checked out, I have full control over my local copy and can do anything to it.  I can change anything and everything about that repository, including the code and any metadata about that repository, including who made the change to a given file or set of files.

Because of this distributed nature of DVCSs, we need a mechanism for merging our various copies of a repository and any respective changes into the upstream repository for a given project.  This, in the open-source git world, is often familiarly facilitated by [GitHub](https://github.com/) in the form of a [pull request](https://help.github.com/articles/about-pull-requests/).  [GitHub](https://github.com/) allows us to manage role-based access on a given project which is bound by that user's [GitHub](https://github.com/) account and their associated SSH private key that grants the permission to push changes to the repo as that user.  This is essentially what prevents me from pushing a change directly to the [Ruby-lang](https://github.com/ruby/ruby) repository, because I do not have [GitHub](https://github.com/) user permissions to push to that repo.

However, due to the nature of DVCSs where I have full control over the repo, I can really send any change I would like to the repo via a PR.  As we've noted with a typical pull request, we often just change files, but what if I wanted to re-write history on the repo, is that possible to offer up as a pull request?  The answer is a resounding **Yes!**, and it's less than forth-coming in the pull request that such actions are happing to the casual PR reviewer.

**Passing the Blame**

So, say for instance I wanted to create a project called [test-repo](https://github.com/claudijd/test-repo).  In [test-repo](https://github.com/claudijd/test-repo) there are a couple of files, which were committed by me ([claudijd](https://twitter.com/claudijd/)) in [pull request #1](https://github.com/claudijd/test-repo/pull/1).

Let's say that I made some huge blunder in [pull request #1](https://github.com/claudijd/test-repo/pull/1) on one of the files, which is already merged into master, and I want to pass the blame to (BuffaloBill).  What I would do is make an ammendment to the git history with the command `git commit --amend --author="BuffaloWill <realuser@realdomain.com>"`.  In order to somewhat hide this fact, I would make legitimate contribution to the "test-repo" project by say updating their README file and sending the aggregate of those changes in [pull request #2](https://github.com/claudijd/test-repo/pull/2).  Your average code reviewer of a PR is probably not going to pick up on the [nuance merge commit](https://github.com/claudijd/test-repo/pull/2/commits) on the PR and skip straight to the [file change tab](https://github.com/claudijd/test-repo/pull/2/files), where all they will review the changes to the README.

This great README contribution that you have made will eventually be merged, assuming that the mystery meta-data change about the prior commit was not picked up on, and the author modification will land in master.

{% highlight bash %}
$ git log --author="Buffalo"
commit b8390d51ac47317435da247cf36346b8c20175b9
Merge: 4584e6b 5bccc6d
Author: BuffaloWill <realuser@realdomain.com>
Date:   Tue Jul 11 22:18:58 2017 -0400

    Merge pull request #1 from claudijd/add_files
    
    Add some files
{% endhighlight %}


**Stealing Credit**

It's not worth really re-stating the above, but basically this works both ways, because you have full control over the content of your local repo, you can also assert that you were person who made a change to said file and somewhat hide that modification in a similar but different legitimate PR contribution.

I can imagine this might be more lucrative if one was incentivized by having a specific set of commits tied to their username or someone wanted to lend credability to a malicious commit by masking it with a more well known contributor.  Perhaps this could even be abused to pepper/corrupt ones contribution graphs in GitHub?

**Solutions**

Now it's only fair of me to offer up some sort of recompense for highlighting such attacks and this mainly comes down to two recommendations...

- Leverage the [GPG Commit verification capability](https://github.com/blog/2144-gpg-signature-verification) in GitHub (arbitrary users cannot edit commits you have signed without breaking their verified status)
- Excercise diligent PR review (pay special attention to PRs with multiple or async commits between PR author and commit author to ensure they are intended changes)

Fundamentally, this behavior is just how DVCSs work and it will not likely be changing anytime soon.  This is because there are legitimate use cases where a Git user may want/need to re-write history and due to the distributed nature, we can all propose those types of changes.  One specific example that comes to mind is when I accidentally landed commits under my personal email address rather than my work email address and we needed to re-write the commit history to properly reflect my contribution as a staff member.

Anyways, I hope you found this little DVCS detour interesting, let me know if you have other creative ideas on other DVCS meta-data abuses and I'll be sure to acknowledge them and credit you in an update on this post.