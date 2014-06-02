---
layout: post
title: "Facebook Graph API v2.0: Building 'Find Friends' and Related Features"
date: 2014-06-02 10:03
comments: true
categories:
- Facebook
- Graph API v2.0
- Find Friends
- Javascript SDK
- APIs
---

{% img left /images/graph_visualization.jpg 300 Graph Visualization %}
In an [earlier post](/blog/changes-in-facebook-graph-api-2-dot-0/), I detailed  some important differences between version 2.0 of Facebook's graph API and the old version.
These include the replacement of global user ids with application-specific ids, and the new restriction of friend data to friends that have connected to the client application.
If you're not familiar with these differences but would like to read on, I recommend reading [that post](/blog/changes-in-facebook-graph-api-2-dot-0/) first.
In this post, I turn my attention to building features with the restricted data that v2.0 affords us.
I will walk through three related features:

1. Finding your users' Facebook friends who are already using your app.
2. Notifying existing users of your app when their Facebook friends register.
3. Allowing existing users to invite their Facebook friends to join.

<!-- more -->


### Find Your Friends

One common use case for integrating with Facebook is to allow users of your application to find friends who have already signed up.
This kind of feature can allow new social platforms to bootstrap community by making use of the connections its users have already formed on Facebook.

The first step is to collect the data that will enable your app to make these connections.
When a user connects to Facebook via your application, you should store the Facebook id for that user.
When the time comes to search for Facebook friends, make a call to `user/friends`, and cross reference the returned user ids with the ones in your database.

Keep in mind that in the new version of the API, you'll only get back friends who have previously connected to your application.
If a user has a Facebook friend who signed up for your service but never connected via Facebook, you will not be able to connect the two of them.
You cannot, for instance, retrieve the email addresses of a user's freinds.

This sounds simple enough, but there's an obvious worry here.
When I recently built a feature like this, I had a <a href="http://media.giphy.com/media/citBl9yPwnUOs/giphy.gif" target="_blank">moment of terror</a>.
The webiste I was working on had been storing Facebook ids for years, but had been doing so under the v1.0 scheme.
That meant that all of the ids we'd stored were global ids.
But now we were upgrading to v2.0, and v2.0 returns app-specific ids.
There is no way to map app-specific ids to global ids; the whole idea of the restrictions imposed by the new API is to prevent applications from sharing information about users.
So how would I ever be able to match up users with their friends?

It turns out that this is a non-issue.
However, to figure that out you need to find the following sentence, which is <a href="https://developers.facebook.com/docs/apps/upgrading/#upgrading_v2_0_user_ids" target="_blank">burried in the Facebook developer docs</a>:

> No matter what version they originally used to sign up for your app, the ID will remain the same for people who have already logged into your app.

In other words, existing v1.0 users of your app get grandfathered in and are still returned from API calls with global ids.
All of those ids stored in the database are still useful for cross-referencing.
Phew!

### Get Notified when Friends Sign Up

After a user has found their existing friends in your application, you may want to inform them when other Facebook friends sign up.
This can also be accomplished using the `user/friends` endpoint.
When a user connects via Facebook, retrieve their (application-scoped) friends, and match their ids up with those in your databse.
You can notify the matched users as you see fit.

### Invite Your Friends to Join

In order to attact new people to your application, you may want to allow existing users to invite their friends via Facebook messages.
This can be accomplished via the <a href="https://developers.facebook.com/docs/sharing/reference/send-dialog" target="_blank">send dialog</a> made available through the Javascript SDK (as well as other SDKs).
However, there are some limitations to how streamlined you can make the inviting process, because you can't really identify the friends your users want to invite. 

Let me explain.
We cannot use the `user/friends` endpoint to find friends for a user to invite, because the only friends it will return are friends who've connected already, and there's no point in inviting  someone to a party they're already at.
There are two endpoints of the new Graph API that will return information about *all* of a user's friends: <a href="https://developers.facebook.com/docs/graph-api/reference/v2.0/user/taggable_friends" target="_blank">`user/taggable_friends`</a> and <a href="https://developers.facebook.com/docs/graph-api/reference/v2.0/user/invitable_friends" target="_blank">`user/inviteable_friends`</a>.
The latter is retricted for use by Facebook games, and so in general, `taggable_friends` is your only option for retrieving a list of all of a user's friends.

However, `taggable_friends` only provides full names, thumbnail profile pictures, and an identifier that's only usable for tagging people in stories published on Facebook.
There is no way to retrieve any kind of useful identifying information, such as a Facebook id (either global or app-specific), an email address, or a Facebook username.
The send dialog requires a Facebook id or username to identify a recipient.
Therefore, *it's impossible to build an 'Invite' button that brings up a Facebook message dialog with recipients pre-filled*.

The best solution I've identified is to have a generic invite button that brings up a dialog with the 'To' field blank.
Using the Javascript SDK, the code to do that looks like this:

```javascript
FB.ui({
    method: 'send',
    link: 'http://join.mywebsite.com'
});
```

If you like, you can inspire your users to invite their friends by showing them profile pictures retrieved from `taggable_friends`.
Here's an mock-up of a feature like that (as designed by <a href="(http://ryanmerrill.net/" target="_blank">Ryan</a> and <a href="http://ryanahamilton.com/" target="_blank">Ryan</a>):

![A grid of Facebook friend profile pictures](/images/Invite Your Facebook Friends.png)

This generic invite functionality is easy to build, and probably nicer for your users than providing the ability to sift through hundreds of Facebook friends.
