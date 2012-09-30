---
layout: post
title: "A tiny benchmarking utility"
date: 2012-09-30 14:54
comments: true
categories: Ruby
---

While working on a longer (forthcoming) post, I wrote a little benchmakring utility that I want to share.
Here it is:

```ruby tiny_timer.rb
module TinyTimer
  def self.benchmark(samples=10)

    times = []

    samples.times do
      start = Time.now
      yield
      times << (Time.now - start)
    end

    times.sum / samples.to_f

  end
end
```

<!-- more -->

As you can see, this method runs a specified number of trials of a block of code and returns the average time in seconds it took to execute the block.
Usage examples:

```bash
>> require 'tiny_timer'
=> ["TinyTimer"]
>> TinyTimer.benchmark{ (0..100000).map{rand(10)}.sort }
=> 0.0670641
>> TinyTimer.benchmark(100){ 2**1000000 }
=> 0.00841815
```
