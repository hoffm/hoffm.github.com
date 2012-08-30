---
layout: post
title: "Quick Tip: More Legible Output with 'Kernel#y'"
date: 2012-08-30 14:24
comments: true
categories: QuickTip Ruby Rails YAML
---

Every tried to get a sense of an ActiveRecord object by squinting though output like this?

```
>> User.find_by_login("Michael Hoffman").recipes.last
=> #<Recipe id: 18891, title: "Pasta with Lemon-Parmesan Butte
r", created_at: "2012-08-29 00:03:55", updated_at: "2012-08-29
16:20:58", description: "I was walking by a little pizzeria in
the East Vill...", recipe_category_id: 36, serving_size: "4 as
a pasta course, 2 as a main", user_id: 784, editors_pick: fals
e, editors_comments: "", label: "pasta_with_lemonparmesan_butt
er", flag_inappropriate: false, wine_id: nil, recipe_style: "a
b", varietal_id: nil, rating_average: 0, makes_serves: "Serves
", flag_inappropriate_comments: nil, sommelier_comments: nil, 
comments_count: 1, is_winner: false, interest: 1, is_finalist:
nil, new_categories_added: nil, admin_categorization_note: nil
>
```
Yuck.

<!-- more -->

Luckily, there's a super simple way to get some nicer formatting.
`Kernel` has a private instance method `#y` which will print out an active record object (or anyting else) as YAML.
Check it out:

```
>> y User.find_by_login("Michael Hoffman").recipes.last
--- !ruby/ActiveRecord:Recipe 
attributes: 
  sommelier_comments: 
  recipe_style: ab
  updated_at: 2012-08-29 16:20:58
  admin_categorization_note: 
  is_finalist: 
  is_winner: "0"
  editors_comments: ""
  serving_size: 4 as a pasta course, 2 as a main
  title: Pasta with Lemon-Parmesan Butter
  interest: "1"
  editors_pick: "0"
  recipe_category_id: "36"
  rating_average: "0"
  id: "18891"
  new_categories_added: 
  flag_inappropriate_comments: 
  wine_id: 
  description: "I was walking by a little pizzeria in the East 
    Village - one I've never been to and may never go to - and
    I slowed my gait to glance at the menu. My eyes happened t
    o land on a pasta dish. I don't remember exactly what it w
    as called, but it involved lemon and parmesan, and not muc
    h else. Lemon! So obvious. Why had I never thought of that
    ? The next night I made this for dinner."
  comments_count: "1"
  makes_serves: Serves
  varietal_id: 
  user_id: "784"
  label: pasta_with_lemonparmesan_butter
  flag_inappropriate: "0"
  created_at: 2012-08-29 00:03:55
=> nil
```

Notice that, in addition to producing more legible formatting, `#y` does not truncate values, so we get to see the whole description of the recipe.
This method is especially useful for visualizing nested hashes and arrays.

```
>> a = {:animals => {:mammals => [:dog, :cat, :narwhal], :fish => [:tuna, :shark] }, :fruits => [:plum, :peach, :banana]}
=> {:fruits=>[:plum, :peach, :banana], :animals=>{:mammals=>[:dog, :cat, :narwhal], :fish=>[:tuna, :shark]}}
>> y a
--- 
:fruits: 
- :plum
- :peach
- :banana
:animals: 
  :mammals: 
  - :dog
  - :cat
  - :narwhal
  :fish: 
  - :tuna
  - :shark
=> nil
```

One warning:
[Apparently](http://stackoverflow.com/questions/11571801/rails-console-y-helper-returns-nameerror-rather-than-yaml-formatting-output/11572045#11572045) this method relies on the [Syck YAML parser](http://stackoverflow.com/a/11572045).
For this reason, it won't immediately work out of the box with Ruby 1.9.3, which uses the [Psych parser](https://github.com/tenderlove/psych/) by default.
If you're using 1.9.3 and you want to play around with `Kernel#y`, run `YAML::ENGINE.yamler = 'syck'` to change parsers.
