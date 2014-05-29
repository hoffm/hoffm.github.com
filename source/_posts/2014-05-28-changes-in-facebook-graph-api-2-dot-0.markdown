---
layout: post
title: "Facebook Graph API v2.0: What's Changed?"
date: 2014-05-29 15:00
comments: true
categories: facebook graph api
---

{% img left /images/Open_Graph_protocol_logo.png Open Graph Logo %}
Facebook recently introduced major changes to its [Graph API](https://developers.facebook.com/docs/graph-api/reference/v2.0).
Despite the existence of an official [upgrade guide](https://developers.facebook.com/docs/games/migrate), these changes have caused
[a](http://stackoverflow.com/questions/23400204/get-facebook-friends-with-graph-api-v-2-0/)
[lot](http://stackoverflow.com/questions/23435961/fetching-list-of-friends-using-facebook-2-0)
[of](http://stackoverflow.com/questions/23836869/facebook-request-dialog-returning-invalid-user-ids)
[confusion](http://stackoverflow.com/questions/23417356/facebook-graph-api-v2-0-me-friends-returns-empty-or-only-friends-who-also-use-m)
for some developers, myself among them.
In this post, I explore in detail the key differences between v1.0 and v2.0 of the API that cause the most frustration.
In a follow-up post, I will discuss how to use v2.0 to build some common features involving finding and inviting Facebook friends.

<!-- more -->

### Less User Data

The updated Graph API is more restrictive in the data it makes available via the [`/user`](https://developers.facebook.com/docs/graph-api/reference/v2.0/user/) and [`/user/friends`](https://developers.facebook.com/docs/graph-api/reference/v2.0/user/friends/) endpoints.
More specifically, the new API provides data that is scoped to the client application:

* Global user ids have been replaced with application-specific ids.
* Friend data is restricted to friends that have connected to the client application.

I'll address each of these restrictions in turn.

### Application-Specific IDs

In the simpler days of v1.0, when the Graph API reported on a user, the id value it provided was a global id number for that Facebook user.
These values were "global" in the sense that they could be shared between client applications.
In v2.0, all user data provided by the `/user` endpoint and its children use application-specific ids.
These ids are a unique identifier not just of the Facebook user, but of the client-user pair.
This mean that you can't share user ids between applications.
(Though Facebook does [provide an API](https://developers.facebook.com/docs/apps/for-business) for mapping between applications that are part of single organization.)

An important consequence of this fact is that the user ids recognized by the Facebook applications you use for development will not match the ones your production app recognizes.
So, if you store your users' Facebook ids in your database in production, you won't be able to match up users in that database with data returned from the API in development.
That's annoying!

And there's no way around it.
You'll just have to query for a few of your friends from the development app and manually overwrite their stored Facebook ids in your development database.
This will give you a few exemplars to develop against.

### Application-Specific Friends

The `user/friends` endpoint used to return exactly what you'd expect: a list of the user's Facebook friends.
Not so in v2.0!
Instead, you'll get a list of the user's Facebook friends *who have connected to your client application at some point in the past*.
This is an enormous difference.
In v2.0 *there is no way to get a list of rich data about each of a user's friends*.
You can't even get their ids, app-specific or otherwise.

If you're upgrading from v1.0 to v2.0, there's another wrinkle:
If a user has *ever* connected to your app using version 2.0, strict permissions govern the data you get back no matter what version you specify in the API call.
Put another way, *once a user connects using the new API, all v1.0 requests are interpreted by Facebook as v2.0 requests*.
This means you need to be careful when deploying an upgrade to your integration with Facebook.
Once v2.0 login is live, there's no going back for those users who connect with it.

### But Don't Despair!

Facebook made these adjustments to give its users more control over their data.
Under the new regime, Facebook data can't easily be shared between applications.
While this can be frustrating for developers, it's probably a good thing for users. 

More importantly for our purposes: Despite the restrictions, it's still possible to build many common features that make use of this data.
In an upcoming post, I'll walk through several of them, including "finding freinds": surfacing a user's Facebook friends who are already users of your client application.
