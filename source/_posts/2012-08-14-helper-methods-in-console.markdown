---
layout: post
title: "Helper methods in the Rails console? There's a helper for that."
date: 2012-08-14 11:53
comments: true
categories: 
---

So you want to mess around with a helper method in the console?
It's super simple, as I just learned [here](http://stackoverflow.com/questions/151030/how-do-i-call-controller-view-methods-from-the-console-in-rails) from [kch](http://caiochassot.com/).
Basically, Rails gets a bit [yo dawg](http://nextlol.com/images/37181-yo-dawg-hotdawg.jpg) and provides a helper for your helpers—called "helper".

Here's an example. Suppose you have a helper module like the following:

<!-- more -->

{% codeblock cars_helper.rb lang:ruby %}
module CarHelper

  def put_a_car_in(location)
    if location == "your car"
      puts "So you can drive while you drive!"
    end
  end

end
{% endcodeblock %}

Fire up a Rails console, and to create the helper-helper, all you have to do in include the module:

```
>> include RecipesHelper
=> Object

>> helper.put_a_car_in("your car")
So you can drive while you drive!
```

I hope this has been helpful. (Sorry, can't help myself.)

Ok, I'm done.
