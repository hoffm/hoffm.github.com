---
layout: post
title: "404 and 500 Error Pages in Rails"
date: 2013-02-09 13:43
comments: true
categories: [Rails, Ruby]
---

There are many approaches to responding to failed requests in Rails.
Here I offer the way I currently favor setting things up.
I'm hoping that this might start a conversation about best practices;
I'm sure I could do better!

The default way Rails handles requests to which your application does not have a contentful response is to serve up static HTML documents stored in the public directory.
In particular, 500 (server error) and 404 (not found) responses result in the application's rendering `500.html` and `404.html`, respectively.

<!-- more -->

This default approach is solid—it can be relied upon to display the correct pages in the correct conditions— but it does not allow customization of those pages.
These error pages can't execute ruby code, and so they can't use your layouts, can't interact with your data, and can't vary based on conditions such as the time, the user, or the nature of the request.
Such customizations can imporve user experience and reduce bounce rates when things go wrong.

For these reasons, it's tempting to rescue all kinds of exceptions and render full-blown erb templates in response to them.
I think this strategy is reasonable in the case of 404 errors, and here's how I do it:

``` ruby app/controllers/application_controller.rb
class ApplicationController < ActionController::Base

  # custom 404
  unless Rails.application.config.consider_all_requests_local
    rescue_from ActiveRecord::RecordNotFound,
                ActionController::RoutingError,
                ActionController::UnknownController,
                ActionController::UnknownAction,
                ActionController::MethodNotAllowed do |exception|

      # Put loggers here, if desired.

      redirect_to four_oh_four_path
    end
  end

end
```

You can define `four_oh_four_path` and the corresponding action and template however you'd like.
A similar but lighter-weight approach would be to render a template in your `app/view/shared/` directory instead of redirecting to an action.
In any case, we wrap this rescue block in the `consider_all_requests_local` condition, because we'd still like to get an informative stack trace in development mode.

It's tempting to treat 500s analogously, but I think that would be a mistake.
If your application is returning a 500 response, chances are something has gone wrong, and the behavior it's exhibiting is not what you expected.
Given this, who's to say whether or not your fancy custom 500 page will load?
What if the bug is in the very code that catches and responds to errors?

For these reasons, it is much safer to take the default approach to 500 errors.
Don't like your static 500 page?
My suggestion is: make sure that people rarely get to see it.

To get some inspiration for such pages, you could the `/500` path your favorite websites.
<a href="https://github.com/500" target="_blank">Here</a>
<a href="https://twitter.com/500" target="_blank">are</a>
<a href="https://path.com/500" target="_blank">some</a>
<a href="http://www.heroku.com/500" target="_blank">examples</a>.
And
<a href="http://food52.com/500" target="_blank">here is the page</a>
I recently made for Food52.
(Thanks to 
<a href="http://ryanahamilton.com/" target="_blank">Ryan</a>
for finding the image and helping make the page responsive!)
