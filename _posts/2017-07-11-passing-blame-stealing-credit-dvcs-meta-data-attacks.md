---
title: "Passing Blame & Stealing Credit with DVCS Metadata Attacks"
layout: post
---

I got into a lively discussion with the OWASP Maine crew tonight about meta-data attacks against distributed version control systems (DVCS) and it simply sparked an interest in writing this hopefully short little post to show how easy it is to abuse, so here goes...

**The Basic Premise**

DVCSs like Git and Mercural are as the name suggests a distributed version control system.  This means that when I perform a checkout of a given project, I have a fully portable copy of that repository.  Once checked out, I have full control over my local copy and can do anything to it.  I can change anything and everything about that repository, including the code and really any metadata about that repository, including who made the change to a given file or set of files.

Because of this distributed nature of DVCSs, we need a mechanism for merging our various copies of a repository and any respective changes into the upstream repository for a given project.  This, in the open-source git world, is often familiarly facilitated by GitHub in the form of a pull request.  GitHub allows us to manage a role-based access on a given project which is bound by that user's GitHub account and their associated SSH private key that grants the permission to push changes to the repo as that user.  This is essentially what prevents me from committing a change directly to the Ruby-lang repository, because I do not have GitHub user permissions to push to that repo.

However, due to the nature of DVCSs where I have full control over the repo, I can really send any change I would like to the repo.  As we've noted with a typical pull request, we often just change files, but what if I wanted to re-write history on the repo, is that possible to offer up as a pull request?  The answer is yes, and it's less than forth-coming in the pull request that such actions are happing to the casual PR reviewer.

**Passing the Blame**

So, say for instance I wanted to create a project called "test-repo".  In test-repo there are a couple of files, which were committed by me (claudijd) in [pull request #1](https://github.com/claudijd/test-repo/pull/1).

Let's say that I made some huge blunder in pull request #1 on one of the files and I want to pass the blame to (BuffaloBill).  What I would do is make an ammendment to the git history with the command `git commit --amend --author="BuffaloWill <realuser@realdomain.com>"`.  In order to somewhat hide this fact, I would make legitimate contribution to the "test-repo" project by say updating their README file and sending the aggregate of those changes in [pull request #2](https://github.com/claudijd/test-repo/pull/2).  Your average code reviewer of a PR is probably not going to pick up on the [nuance merge commit](https://github.com/claudijd/test-repo/pull/2/commits) on the PR and skip straight to the [file change tab](https://github.com/claudijd/test-repo/pull/2/files), where all they will see is the changes to the README.

This great README contribution that you have made will eventually be merged, assuming that the mystery meta-data change about the prior commit was not picked up, and the author modification will land in master.

**Stealing Credit**

It's not worth really re-stating the above, but basically this works both ways, because you have full control over the content of your local repo, you can also assert that you were person who made a change to said file and somewhat hide that modification in a legitimate PR contribution.

I can imagine this might be more lucrative if one was incentivized by having a specific set of commits tied to their username.  Perhaps this could even be abused to pepper ones contribution graphs in GitHub?

**Solutions**

Now it's only fair of me to offer up some sort of recompensation or mitigating means of preventing such attacks and I'm guessing this mainly comes down to two things...

- Leverage the GPG Commit Signing Capability in GitHub (arbitrary users cannot edit commits you have signed, I think)
- Excercise diligent PR review (pay special attention to PRs with multiple or async commits between PR author and commit author)

Fundamentally though, this is how DVCSs work and we need to live in.  This is because there are legitimate use cases where a Git user may want/need to re-write history.  Once specific example that comes to mind from my development background is when I was working on a commerical product and I accidentally committed code under my personal email address rather than my work email address and we needed to re-write the commit history to properly reflect my contributions.