---
layout: post
title: "Optimizing ActiveRecord Queries"
date: 2012-09-29 11:37
comments: true
categories: ActiveRecord, Rails
---

Here's a common kind of simple programming task:
Create a list of the email addresses of users who've opted to receive the newsletter.
Using the tools provided to us by Ruby, MySQL and ActiveRecord, there are dozens of ways this task could be accomplished.

If your database only contains a few hundred users, it's not that important to retreive this list in the fastest possible way.
However, if you're users table has tens of thousands of rows—or more—the various methods for retrieving this list range from "zippy!" to "browswer timeout".
In this post, I want to go through a series of successively faster ways to accomplish this task.
The series roughly paralells the chronology of my own learning about data base queries, and about the ActiveRecord query interface in particular.

In order to get some actual numbers, I'm going to work with an actual app.
The numbers cited below are derived from performing queries on table with about 85,000 rows, and quite a few columns:

```ruby
>> User.count
=> 84491
>> User.columns.count
=> 79
```

Ok, on with the task at hand.

### But first

### First Attmept

Let's try the most straight forward, obvious approach.
