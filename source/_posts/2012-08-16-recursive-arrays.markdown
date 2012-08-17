---
layout: post
title: "Recursive Arrays in Ruby"
date: 2012-08-16 22:30
comments: true
categories: 
---

They're real!

```ruby
>> a = []
=> []
>> a << a
=> [[...]]
>> a.first.equal? a
=> true
```
<!-- more -->

But [why](http://stackoverflow.com/questions/10606734/what-are-recursive-arrays-good-for)?
