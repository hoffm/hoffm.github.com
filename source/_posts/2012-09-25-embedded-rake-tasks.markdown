---
layout: post
title: "Invoke a Rake Task from Another Task"
date: 2012-09-25 17:02
comments: true
categories: rake Ruby
---

You can invoke a rake task from another rake task.
Here's how:

```ruby examples.rake
namespace :examples do

  desc "Inner task."
  task :inner_task => :environment do
    # Some code that accomplishes a task.
  end

  desc "Outer task."
  task :outer_task do
    # Some code.
    Rake::Task['examples:inner_task'].execute #Execute innter task.
    # Some other code.
  end

end
```

<!-- more -->

Now when you execute `rake examples:outer_task`, you will execute the inner task in whatever context you placed it.
Using this contruction allows you to break down unweildy tasks into smaller ones, which perhaps you'll want to reuse in different contexts elsewhere.

Here's a toy examples that actually works:

```ruby toys.rake
namespace :toys do

  desc "Inner task: Square the value."
  task :square do
    @value *= @value
  end

  desc "Outer task: Initialize the value and square thrice."
  task :square_three_times do
    @value = 2
    3.times{Rake::Task['toys:square'].execute}
    puts "The value is #{@value}."
  end

end
```

Running this in the shell, we get:

```bash
$ rake toys:square_three_times --trace
** Invoke toys:square_three_times (first_time)
** Execute toys:square_three_times
** Execute toys:square
** Execute toys:square
** Execute toys:square
The value is 256.
```

And yes, I know you're thinking it now: rake tasks can call themselves quite happily.

You can also invoke rake tasks from within ActiveRecord migrations.
This can come in handy when you want to initialize a bunch of data immeidately after altering the database schema.
