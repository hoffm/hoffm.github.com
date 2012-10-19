---
layout: post
title: "How do you prevent ActionMailer from spamming people in development?"
date: 2012-10-18 19:09
comments: true
categories: Rails, ActionMailer
---

I've been working on a Rails app that contains lots of mailers with hard-coded recipients.
For instance, here's a mailer that we use to notify folks when someone asks an urgent question on our hotline:

```ruby hotline_mailer.rb
class HotlineMailer < ActionMailer::Base

  default :from => "notifications@mywebapp.com"

  def notify_urgent_question(question)
    @question = question
    send( :to => "urgent@mywebapp.com", 
          :subject => "Urgent: #{question.title}")
  end

  # Other mailer methods...

end
```

The email address "urgent@mywebapp.com" generates emails to many people.
We handle membership in Google apps.

Left to its own devices, this mailer will trigger an email to that list of people every time you hit the controller action that calls for it to be delivered.

That's bad news if you're just testing out the action in your development environment.
You have a group of people who are getting annoying emails for no reason, and you may not even be getting the emails yourself!

<!-- more -->

We might consider using a constant in place of the email address, and initializing this constant differently in different environments.
However, this approach has two shortcomings:

 1. If we have more than one person working on the project, it's hard to choose which email address to use as the default for development.
 2. We'll all have to remember to make a new constant every time we have a new email address that needs to be masked.


Luckily, ActionMailer makes a better solution possible. Here's what you need to do:

```ruby lib/development_mail_interceptor.rb
class DevelopmentMailInterceptor

  def self.delivering_email(message)
    message.to = defined?(PERSONAL_EMAIL) ? PERSONAL_EMAIL : "default@mywebapp.com"
  end

end
```

```ruby config/initializers/mailer_config.rb
require 'development_mail_interceptor'
ActionMailer::Base.register_interceptor(DevelopmentMailInterceptor) if  Rails.env.development?
```

```ruby config/initializers/personal_email.rb
PERSONAL_EMAIL = "personal@mywebapp.com"
```

Lastly, you want to git ignore `config/initializers/personal_email.rb`, and tell your collaborators to create their own local version of the file.
(You want to git ignore it so that it can be customized per dev box.)

What does this all do?
When you call `#deliver` on an message, if the environment is development, the `to` option on the mailer is overridden with your personal address.
(In this example, it's "personal@mywebapp.com").
That's it!
We set up the default in the interceptor in case one of our collaborators doesn't get the message about creating `personal_email.rb`.
In that case, that person's dev mailer emails will be sent to the default you set in the interceptor class.
(In this example it's "default@mywebapp.com")

When I first struck upon this solution, I was worried that the interceptor would prevent me from seeing who would actually receive a given email if the code were running in production.
I needn't have worried.
The interceptor only affects the behavior of the `#deliver` method; the message object remains intact.
So, if you call

> `HotlineMailer.notify_urgent_question(question)`

you'll find that object returned has the attribute `<To: urgent@mywebapp.com>`.
That's the address this email would go to if the interceptor did not interfere.
However, if you call

> `HotlineMailer.notify_urgent_question(question).deliver`,

the object returned has the attribute `<To: personal@mywebapp.com>` and sends the email to that address.

I think this is an effective solution, but it could be better.
Forcing developers to manually create files in order to properly use your repo is not ideal.
If you have an idea to improve this system, I'd love to hear about it in the comments.
