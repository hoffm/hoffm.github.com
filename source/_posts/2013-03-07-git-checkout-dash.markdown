---
layout: post
title: "How to checkout the last git branch you were on"
date: 2013-03-07 11:10
comments: true
categories: git QuickTip
---

Checking out the last branch you were on is as simple as `git checkout -`.

``` bash
11:13:14-hoffm~/src/food52 (develop)$ git checkout master
Switched to branch 'master'
11:13:21-hoffm~/src/food52 (master)$ git checkout -
Switched to branch 'develop'
11:13:23-hoffm~/src/food52 (develop)$ git checkout -
Switched to branch 'master'
```

[Boom](http://i.imgur.com/yUNhh.gif)!
