---
layout: post
title: "All your undefined vars are evaluate to nil."
date: 2012-08-27 18:14
comments: true
categories: 
---

*(NB: I learned much of what I discuss here from this [post](http://stackoverflow.com/a/4023224/882025).)*

What does a brand new, never-before-mentioned variable evaluate to in Ruby?
Perhaps it seems that the correct answer is: it depends what kind of variable.
Undefined instance variables, for example, evaluate to `nil`, where as undefined local variables can't be evaluated at all.

<!-- more -->

A little experimentation in irb seems to confirm this hypothesis.
Uninitialize local variables raise a `NameError`:

```
>> local_variables
=> []
>> local_var
NameError: undefined local variable or method `local_var' for main:Object
```
whereas instance variables evaluate to `nil`:

```
>> instance_variables
=> []
>> @instance_var
=> nil
```

(Global variables behave the same way in this respect as instance varables.)

So, again, it seems as if the correct answer to our question is, "It depends."
But that's wrong.
In fact, the question has a single, simple correct answer:
*All types of uninitialized variables evaluate to nil.*

Here's proof:

```
>> local_var = "something or other" if false
=> nil
>> local_var
=> nil
```

The initialization of `local_var` is never evaluated, because the condition `false` is, of course, not met.
Yet, it seems that the mere fact that the parser has parsed `local_var` suffices to avoid the `NameError` when evaluating the variable.
Why?

The answer is that the `NameError` was merely an error of ambiguity:
In our first example, the parser could not decide whether `local_var` was a local variable, or the name of a method—shorthand for `self.local_var`.
The error message we got didn't do a great job of explaining this, but it did hint at it with the phrase "variable or method".
Once the parser has encountered `local_var` in an unambiguously variable-like context—namely, `local_var = "something or other"`— it has no problem evaluating the name as the name of a variable.
And, as I've claimed, all uninitialized variable evaluate to `nil`.

So, what appeared to be a difference between local varables and other types of variables is actually just a coincidence of Ruby syntax.
Ruby gives us the flexibility to leave the receiver to which we pass a method implicit:
if we don't explicitly identify a reciever, the message is passed to the current object.
However, this flexibilty comes at the price of ambiguity;
absent additional context, methods invoked on implcitly defined receivers look to the world (and, more importantly, the parser!) just like local variables.
