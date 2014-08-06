---
layout: post
title: "Facebook Graph API v2.0: What's Changed?"
date: 2014-05-29 13:41
comments: true
categories:
- Facebook
- Graph API v2.0
- Application-Specific IDs
- APIs
---

{% img left /images/Open_Graph_protocol_logo.png Open Graph Logo %}
Facebook recently introduced major changes to its <a href="https://developers.facebook.com/docs/graph-api/reference/v2.0" target="_blank">Graph API</a>.
Despite the existence of an official <a href="https://developers.facebook.com/docs/games/migrate" target="_blank">upgrade guide</a>, these changes have caused
<a href="http://stackoverflow.com/questions/23400204/get-facebook-friends-with-graph-api-v-2-0/" target="_blank">a</a>
<a href="http://stackoverflow.com/questions/23435961/fetching-list-of-friends-using-facebook-2-0" target="_blank">lot</a>
<a href="http://stackoverflow.com/questions/23836869/facebook-request-dialog-returning-invalid-user-ids" target="_blank">of</a>
<a href="http://stackoverflow.com/questions/23417356/facebook-graph-api-v2-0-me-friends-returns-empty-or-only-friends-who-also-use-m" target="_blank">confusion</a>
for some developers, myself among them.
In this post, I explore in detail the key differences between v1.0 and v2.0 of the API that cause the most frustration.
In a [follow-up post](/blog/find-friends-with-facebook-graph-api-2-dot-0/), I discuss how to use v2.0 to build some common features involving finding and inviting Facebook friends.


<!-- more -->

### Less User Data

The updated Graph API is more restrictive in the data it makes available via the <a href="https://developers.facebook.com/docs/graph-api/reference/v2.0/user/" target="_blank">`/user`</a> and <a href="https://developers.facebook.com/docs/graph-api/reference/v2.0/user/friends/" target="_blank">`/user/friends`</a> endpoints.
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
(Though Facebook does <a href="https://developers.facebook.com/docs/apps/for-business" target="_blank">provide an API</a> for mapping between applications that are part of single organization.)

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
In [my next post](/blog/find-friends-with-facebook-graph-api-2-dot-0/), I walk through several of them, including "finding friends": surfacing a user's Facebook friends who are already users of your client application.

---

**Update, May 31, 2014**: <a href="https://twitter.com/EmilHesslow" target="_blank">Emil Hesslow</a>, an engineer at Facebook, has pointed out to me that the "Test App" feature, which was introduced on April 30, 2014 along with the new version of the API, and which allows you to nest development applications under a production application, has a nice property that I glossed over:

> If you create a new Test app (a feature we added in v2.0) for your production app they will share the same app scoped user ids. You can read about them <a href="https://developers.facebook.com/docs/apps/test-apps" target="_blank">here</a>.

Although I'd been using this feature, I hadn't realized that it allowed test applications to share ids with production applications.
My production app was still running under v1.0, so I had global ids stored in my production database.
However, my test application was running v2.0 (as I was working on the upgrade) and received app-specific ids from the API.
Thus I could not develop against a dump of the production database without manually altering the Facebook ids that were stored for my application's users.

I actually think my case is by far the most common one, for now at least.
The vast magority of Facebook applications were created before April 30th, 2014, and are therefore governed by pre-v2.0 rules.
However, all nested test apps were necessarily created after that date, which leads to the id mismatch.

---

**Update, August 6, 2014**: <a href="https://twitter.com/sicross" target="_blank">Simon Cross</a>, Platform PM at Facebook, has alerted me to an inaccuracy in the update above: He says that test applications "inherit the version availability of their parent". Presumably this means that no matter when the test app is created, if its parent app has been grandfathered into v1.0, then the test app will be too. This behavior is detailed in <a href="https://developers.facebook.com/docs/apps/test-apps" target="_blank">Facebook's documentation on test apps</a>, which is much more detailed than it was at the time I wrote the original post.
