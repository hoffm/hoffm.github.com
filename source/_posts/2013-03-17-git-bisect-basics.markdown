---
layout: post
title: "Git Bisect: Your Friend in Times of Need"
date: 2013-03-17 11:38
comments: true
categories: [git, debugging]
---

Git bisect is like CPR or a fire extinguisher:
You shouldn't need it very often, but when you, it might just save a life.

In case you are not familiar with its splendor, git bisect is a tool for pinpointing the commit that caused a change.
Usually, the change you're interested in is the introduction of a bug.
Bisect is handy when, for instance, you discover a regression and you don't know what caused it or exactly how long it's been around.
Bisect isolates the offending commit with logarithmic efficiency.
It accomplishes this by performing a binary search of the commits on your branch, dividing and conquering until only one commit remains.

<!-- more -->

The basic version of the process is simple.
You provide git with a good commit (one you know is after the change) and a bad commit (one you know is before the change).
Git then asks you a series of yes-or-no questions of the form: How about this other commit here—is it good?
Approximately log-base-2 of the number of questions in your good-bad interval later, you have your culprit.

Here are the commands for navigating through this process:

```bash
$ git bisect start
    # "Let's do this thing."
$ git bisect bad
    # "Hey, git, the current commit bad."
$ git bisect bad [commit hash or tag name]
    # "This commit hash refers to a bad commit."
    # (Only one of the previous two commands is required.)
$ git bisect good [commit or hash]
    # "Here's an earlier point at which I know things were good."
```

At this point, Git will proceed to checkout commits for you.
Your job is to give the thumbs up or thumbs down, Roman-emperor style.

```bash
$ git bisect good
    # "This commit is before the change; look later."
$ git bisect bad
    # "This commit is after the change; look earlier."
```

At each step you'll need to do whatever it is you need to do to check which side of the change the commit is on.
If you're working on a Rails app, this might involve running `rake db:migrate` and `bundle install`, and then checking whether a particular test passes or a particular behavior of your application is present.

When git zeroes in on its target, it will spit out a message telling you what it found, e.g.

```bash
341c8a0f26e67cf8735552cf8f2b945b80ccb84b is the first bad commit
commit 341c8a0f26e67cf8735552cf8f2b945b80ccb84b
Author: hoffm <m@hof.fm>
Date:   Thu Mar 17 11:19:10 2013 -0500

    Draft of git bisect post.
```

At this point, you'll probably want to get out of bisect mode and back onto your working branch.
To do this, run

```bash
$ git bisect reset
```

That's all you really need to know to start using git bisect.

In researching this post, though, I came across some additional commands that are worth learning.
First, if during the bisect process, git lands you on a particularly awkward commit—e.g. one for which the needed environment is hard to replicate from your present position—you can run `git bisect skip` and git will offer you a different commit to test.

Second—and way cooler—you can automate your yes/no decisions by feeding a custom script to `git bisect run`.
Git will run the script at every decision point, and will take an exit code of 0 to mean good, and anything else (except 125) to mean bad (details [here](https://www.kernel.org/pub/software/scm/git/docs/git-bisect.html#_bisect_run)).
Test frameworks like RSpec [exit with 0](https://www.relishapp.com/rspec/rspec-core/v/2-13/docs/command-line/exit-status) only when all tests pass (or none are run), so if you're trying to identify the commit that caused a test failure, this is your tool.
