---
layout: post
title: "Try out Routes in the Rails Console"
date: 2013-02-04 20:23
comments: true
categories: Rails, Ruby, QuickTip, routing
---

How do you quickly check what path a certain route helper maps to?

One method I use a lot is to grep `rake routes`.
So, let's say I'm looking for a route that has to do with comment approval.
In that case, I might run `rake routes | grep approve`.
And there's my answer:

``` ruby
approve_comment POST  /comments/:id/approve(.:format)  {:action=>"approve", :controller=>"comments"}
```

There's one problem with this, though: It's
  <a href="http://i.minus.com/iBKc6u9W2Zreb.gif" target="_blank">slooooow</a>.

<!-- more -->

It's slow because running `rake routes` loads up your whole Rails environment.
If your workflow is anything like mine, though, chances are you've already have your environement loaded in another tab: the one where you have a Rails console running.
So in cases where you have some idea what the name of the route is, testing it  out in the console can save you time.
And, luckily, it couldn't be simpler.

In the Rails console, the variable `app` holds a session object on which you can call path and url helpers as instance methods.
So away we go:

``` ruby
>> app.approve_comment_path(Comment.last.id)
=> "/comments/138138/approve"
>> app.user_path(User.find_by_login("Michael Hoffman"))
=> "/users/784-michael-hoffman"
```

That's all there is to it.

*(See [this post](code-worrier.com/blog/custom-slugs-in-rails/) for my way of generating custom slugs like the one in that user path.)*

*(Searching around a bit, I think I originally got this tip from <a href="http://37signals.com/svn/posts/3176-three-quick-rails-console-tips" target="_bank">this wonderful post</a>. Read it to learn about other useful things you can accomplish with `app`.)*
