---
layout: post
title: "git bisect basics"
date: 2013-03-03 11:05
comments: true
categories: git, debugging
---

Git bisect is like CPR or a fire extinguisher:
You shouldn't need it very often, but when you do need it it might just save a life.

In case you are not familiar with it's splendor, git bisect is a tool for pinpointing the commit that caused a change.
Usually, the change your interested in is a bug.
This is handy when, for instance, you discover a regression and you don't know what caused it or exactly how long it's been around.
Bisect isolates the offending commit with logarithmic efficieny.
It accompishes this by performing a binary search of the commits on your branch, dividing an conquering until only one commit remains.

The basic version of the process is simple.
You provide git with a good commit (one after the change) and a bad commit (one you know is before the change).
Git then proceeds to ask you a series of yes-or-know questions of the form: How about this commit here—is it good?
Approximately log-base-2 of the number of questions in your good-bad interval later, you have your culprit.

Here are the actual commands:

```git
$ git bisect start # Let's do this thing.
$ git bisect bad # Hey, git, the current commit is the earliest bad commit I know.
$ git bisec bad [commit hash or tag name] # Here's the earliest bad commit I know.
$ git bisect good [commit or hash] # Here's an earlier point at which I know things were good.
```

Git will proceed to checkout commits for you.
Your job is to give the thumbs up or thumbs down.

```git
$ git bisect good # This commit is before the change; look later.
$ git bisect bad # This commit is after the change; look earlier.
```

That's all there is too it.
At each step you'll need to do whatever it is you need to do to check which side of the change the commit is on.
If you're working on a Rails app, this might involve running `rake db:migrate` and `bundle install`, and then checking whether a particular test passes or a particular behavior of your application is present.

That's all you really need to know to use git bisect.
In researching this post, I found some other cool commands that seem pretty useful.
For instance, if durring the bisect process, git lands you on a particularly awkward commit—e.g. one for which the needed environment is hard to replicate from your present position—you can run `git bisect skip` and it will offer you a different one.

Even cooler than that: you can automate your yes/no decisions using `git bisect run` and feeding it a script.
Git will run the script at every decision point, and will take an exit code of 0 to mean good, and anything else (except 125) to mean bad.
Luckily, `TestCase` examples exit with 0 when they pass and 1 when they fail.

What git bisect does.

How to use it.

Start, good, bad.

Run with test case
