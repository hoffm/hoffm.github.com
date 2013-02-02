---
layout: post
title: "Custom Slugs in Rails without Gems"
date: 2013-02-01 16:48
comments: true
categories: [Ruby, Rails]
---


There are some
 <a href="https://github.com/norman/friendly_id" target="_blank">nice</a>
 <a href="https://github.com/Sutto/slugged" target="_blank">gems</a>
out there to help you create and manage human-readable slugs in Rails.
However, our requirements at Food52 were simple enough—and different enough from the standard case—that I thought I'd have a go at building a slugging system from scratch.

## The Problem

As an example of the desired behavior, given that we want to use the attribute `name` to generate slugs for users, and given that my user id is 784 and my name is "Michael Hoffman", the slug for my user page should be:

> `"784-michael-hoffman"`

(We leave the id in because we like it there!
It makes debugging easier and it allows us to use Active Record `find` to look up objects from params.)

Furthermore, we want our slugs to adapt gracefully to changing circumstances.
So, if my name changes, the slug should do...something smart.

Getting a little more abstract, our requirements are as follows:

<!-- more -->

  1. It should be possible to opt models in to custom slugging.
  2. Custom slugs should have the form: `[:id]-[parameterized-string]`.
  3. The string to be parameterized should be configurable per model.
  4. Relevant changes in records' attributes should be reflected in the slug.
  5. Records should have only one slug apiece, even as attribute values change.

That's the problem.
In this post, I explain my solution.
Even if your custom slug needs are slightly different, I hope the general strategy might be useful.


## A Naive Attempt

Before we begin, we need some background on how Rails handles slugs by default.
Active Record has an instance method `#to_param`, which Action Pack uses to create URLs.
By default, `#to_param` returns the id of the object, converted to a string.
(See the source,
 <a href="http://apidock.com/rails/ActiveRecord/Integration/to_param" target="_blank">here</a>
.)
Thus, the default path to my user page (given standard resourceful routing) would be `"/users/784"`.
However, it is possible to customize slugs by overriding this method on a model-by-model basis.

So, the naive way to get the result we want would be to add a method like this one to the User model:

``` ruby app/model/user.rb
class User < ActiveRecord::Base

  # [...]

  def to_param
    [id, name.parameterize].join("-")
  end

end
```

This approach meets the first four requirements, but there are two problems with it.
First, we're going to have to repeat this method (modulo the string to be parameterized) in all the models for which we want human-readable slugs.

Second, it fails to meet the fifth requirement.
Specifically, as users' names change, we will end up with many different addresses that point to the same content.
For instance, if I change my name to, say, "Michael Danger Hoffman", then Action-Pack-generated links to my profile will use `784-michael-danger-hoffman`, but anyone who'd previously copied the link, or bookmarked it, will be able to reach the same page by visiting the old slug.
Not only is that confusing to humans, it's also confusing to search bots.
Confusing search bots makes them <a href="http://www.geekologie.com/2011/09/22/stupid-robots.gif" target="_blank">sad</a>!

## A Better Solution

Let's build on the naive solution to create a slugging system that can be applied succinctly across many models.
(We'll deal with the duplicate content problem in a another pass.)

Given a model, there are only two things we need to know about it with regard to slugging:

 * Should it use custom slugs?
 * If so, what string should it use to create them?

Ideally, we want a system that would allow us to opt models in to custom sluging, and simultaneously to specify how to calculate the strings those models should use to create slugs.

We can do this by adding a class method to `ActiveRecord::Base` that redefines `#to_param`.

``` ruby lib/app_utilities.rb
module ActiveRecordExtensions
  extend ActiveSupport::Concern

  module ClassMethods

    # Override 'to_param'.
    def custom_slugs_with(seed)
      self.redefine_method(:to_param) do
        [id, self.send(seed).parameterize].join("-")
      end
    end

  end

  # Include above methods in all models.
  ActiveRecord::Base.send(:include, ActiveRecordExtensions)

end
```

We're going to want to make `::custom_slugs_with` available in all models, so let's require this file in an initializer:

``` ruby config/initializers/active_record_extensions.rb
require "app_util"
```

Requiring this file will execute the include call, including `ActiveRecordExtensions` in `ActiveRecord::Base` (andmaking the module's methods available to Active Record's descendants).
The `::custom_slugs_with` method takes as an argument the name of an instance method that specifies the string to be used in the slug.
So, for instance, adding this line to the User model:

``` ruby app/model/user.rb
class User < ActiveRecord::Base
  custom_slugs_with(:name)

  # [...]
end
```

will serve the same purpose as the `to_param` method in our original, naive solution.

When we want to retrieve the object from its slug, as will often be the case in controller actions, we can use a simple Active Record find. E.g.

``` ruby app/controllers/users_controller.rb
class UsersController < ApplicationController

  #[...]

  def show
    @user = User.find(params[:id])
  end

end
```

Find is smart enough to parse strings like these and extract the correct id.

Lastly, if we wanted to use an explicit method to generate the string (instead of relying on the attribute, `name`), we could define that method in the User model and pass its symbolized name into the `::custom_slugs_with`.


## Keeping Slugs Consistent

We still have a lingering problem.
The attributes from which these custom slugs are generated might change.
Moreover, we might be installing this system in an application that already has a different slugging system.
Both of these eventualities will lead us into to the uncomfortable situation I mentioned above: having multiple URLs that refer to the same content.
To get around this problem, let's treat the record's current slug—the one we get when we call `#to_param` on it—as canonical.
Then, when we get a request for an out-of-date or incorrect slug, we can redirect the canonical one.

We'll do this in basically the same way in every controller, so let's add our new code to the application controller.
The first method checks if the slug is canonical.
The second redirects to the canonical slug, as retrieved by Action Pack.

``` ruby app/controllers/application_controller.rb
class ApplicationController < ActionController::Base

  #[...]

  ##
  # Check if the current slug is not the cannonical one.
  def bad_slug?(object)
    params[:id] != object.to_param
  end

  ##
  # 301 redirect to canonical slug.
  def redirect_to_good_slug(object)
      redirect_to params.merge({
                    :controller => controller_name,
                    :action => params[:action],
                    :id => object.to_param,
                    :status => :moved_permanently
                  })
  end
end
```

Now we can call these methods from controller actions that we want to redirect non-canonical slugs:

``` ruby app/controllers/users_controller.rb
class UsersController < ApplicationController

  #[...]

  def show
    @user = User.find(params[:id])
    redirect_to_good_slug(@user) and return if bad_slug?(@user)
  end

end
```

This gets the job done.
Requests for `"/users/784-something-totally-weird"` will redirect to `"/users/784-Michael-Hoffman"` as desired.
Changing my name will change the target of this redirection.

I must admit, though, that I find this solution less than ideal.
It requires repeating the redirect-if-bad-slug code in many actions.
However, I have not yet found a better approach.
If you have one, I'd love to hear it!
