---
layout: post
title: "Passing Symbols, a Kata Lesson"
date: 2012-08-12 22:02
comments: true
categories: ruby, kata
---

In [my post](http://code-worrier.com/blog/kata/) about the [kata gem](https://github.com/wbailey/kata), I promised to share something I learned from the [calculator kata](https://github.com/wbailey/code_katas/blob/master/calculator.rb).
Here goes.

The calculator kata asks you to construct a `Calculator` class the instances of which are initialized with comma-delimited strings of integers.
Next you are asked, one by one, to create a series of instance methods—`#sum`, `#prod`, `#div`, `#diff`—for the class that perform arithmetic operations on the stored integers.

Starting with `#sum` only, I wrote the class, obviously enough, like this:

<!-- more -->

```ruby
  class Calculator

    def initialize expr
      @expr = expr
    end

    def sum
      @expr.split(/\,/).map(&:to_i).inject(&:+) || 0
    end

  end
```

After adding in another instance method, it made sense to move to conversion from string to array into the initialization method:

```ruby
  class Calculator

    def initialize expr
      @expr_array = expr.split(/\,/).map(&:to_i)
    end

    def sum
      @expr_array.inject(&:+) || 0
    end

    def prod
      @expr_array.inject(&:*)
    end

  end
```

I went on to add analogous subtraction and division methods, and then I started to get an itch:
Could I avoid writing "inject" four times?
More importantly, could I find a way that to write this class that reflected the underlying similarity in function shared by these four methods?

On a kata-fuelled lark, I decided to write the code I *wished* would work, even though I had no reason to think it would.
I passed the arithmetic operation symbols into the methods as arguments, and then into the inject blocks:

```ruby
  class Calculator

    def initialize expr
      @expr_array = expr.split(/\,/).map(&:to_i)
    end

    def sum
      inject_op(:+) || 0
    end

    def prod
      inject_op(:*)
    end

    # etc.

    private

    def inject_op op
      @expr_array.inject(& op)
    end

  end
```

To my surprise *this actually does work*!!

My initial reaction was: Go Ruby!
And then: Thank you, kata.

But, finally, in the spirit of the aphorism that "magic is code you don't understand yet," I thought: How do these `&` blocks work, anyway?
More on that in a later post.
