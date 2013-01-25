---
layout: post
title: "Put Your Git Branch in Your Bash Prompt"
date: 2013-01-24 21:54
comments: true
categories: QuickTip git bash
---

Lots of folks were interested in getting [git autocomplete](http://code-worrier.com/blog/autocomplete-git/) going in bash, so I thought I'd offer another useful git/bash tip.
This one was suggested by [Stephan](http://twitter.com/S_2K) in the comments of that earlier post.

If you're using git properly and collaborating on a project, chances are you find yourself changing branches often.
This can be quite disorienting.
You deploy from the master branch, then get back to that bug fix you were about to work except for—whoops!—now you commited that fix to master and have to [back away slowly](http://d22zlbw5ff7yk5.cloudfront.net/images/cm-23141-050624c7335479.gif).
Confusions like this one can easily be avoided by putting you branch name front-and-center: at the end of your bash prompt.
Here's how.

<!-- more -->

Get the `git-promt.sh` script [here](https://github.com/git/git/blob/master/contrib/completion/git-prompt.sh).
Put it somewhere like `~/.git-prompt.sh`.
If you want to be all CLI about it, you could just run:

``` bash
curl https://raw.github.com/git/git/master/contrib/completion/git-prompt.sh -o ~/.git-prompt.sh
```

Now modify your your bash profile (it's at `~/.bash-profile`, in case you're new to this stuff).
Before the part of the file that declares what your prompt will look like (`PS1=[...]`), load in the script you just downloaded, like so:

``` bash
# Load in the git branch prompt script.
source ~/.git-prompt.sh
```

Now add `\$(__git_ps1)` to your prompt declaration.
You can put it wherever you like, but I think it's best suited for the very end, right before the final dollar sign (`\$`)
If you want to get all fancy, you can make it a pretty color.
My declaration looks like this:

``` bash
PS1="\[$GREEN\]\t\[$RED\]-\[$BLUE\]\u\[$YELLOW\]\[$YELLOW\]\w\[\033[m\]\[$MAGENTA\]\$(__git_ps1)\[$WHITE\]\$ "
```

(See [here](https://wiki.archlinux.org/index.php/Color_Bash_Prompt) for color definitions.)

This declaration results in a prompt looks like this when I'm in the directory `~/src/food52` and on the branch `develop`:

![](http://f.cl.ly/items/0b3e1Q2R1I3b2T2a1q03/Screen%20Shot%202013-01-25%20at%2010.31.45%20AM.png)

That's it. Never git lost again!
