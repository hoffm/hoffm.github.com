---
layout: post
title: "Quick Tip: Recursive Arrays in Ruby"
date: 2012-08-16 22:30
comments: true
categories: Ruby QuickTip
---

They're real!

In Ruby, an array can contain itself:

```ruby
>> a = []
=> []
>> a << a
=> [[...]]
>> a.first.equal? a
=> true
```
<!-- more -->

But [why would you want to go and do that?](http://stackoverflow.com/questions/10606734/what-are-recursive-arrays-good-for)?
