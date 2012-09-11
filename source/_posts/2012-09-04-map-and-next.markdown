---
layout: post
title: "What does Ruby's 'next' keyword return?"
date: 2012-09-04 19:55
comments: true
categories: QuickTip Ruby Rails
---

Pop quiz, hotshot:
**What does the following Ruby expression return?**

```ruby
[1,2,3].map{ |n| next if n.even? ; 2*n }
```

<!-- more -->

It turns out I'm not much of a hotshot, because this how I reasoned through this question while at work today:

  1. `Array#map` evaluates the block for each member of the array and returns a new array of the returned values, in order.
  2. When `n == 1` or `n == 3`, the block evaluates to `2*n`.
  3. However, when `n == 2` we hit the `next` keyword and nothing gets evaluated.
  4. Therefore this expression returns `[2,6]`.

Reinforcing this line of reasoning, a very similar expression,

```ruby
[1,2,3].each{ |n| next if n.even? ; puts 2*n }
```

gives the analog of this result:
it prints "2" and then "6" and then return.
The iteration that hits ```next``` just gets skipped.
Analogously, the version with `map` should return `[2,6]`

**Wrong!**

In fact, the answer to the quiz is... 

```ruby
[2, nil, 6]
```

The error in the reasoning above is contained in premise (3).
It turns out that **when `next` is evaluated, it returns `nil`**.

Similarly,

```ruby
[1,2,3].map{ |n| 2*n if n.odd? }
```

returns the same array, `[2, nil, 6]`, because `... if false` returns nil no matter what (syntactically valid) expression `...` is.

So, if you're using `map`, and you want to skip some elements, you can't just use `next` or `if`.
You also need to deal with the `nil`s when you're done, like this:

```ruby
1,2,3].map{ |n| next if n.even? ; 2*n }.compact
```

Alternatively—although it's probably more expensive—you could select the elements you want before proceeding with the operations you want to perform on them:

```ruby
[1,2,3].reject{ |n| n.even? }.map{ |n| 2*n }
```

Let me know in the comments if you can think of a generalizable implementation that's more elegant than either of these.
In the meantime, may you never make the same mistake I made!
