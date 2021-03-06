---
title: Ember Router v2 Upgrade Notes
date: 2013-01-27
---

{:javascript: class=javascript}


I've been working on an [Ember](http://emberjs.com/) project for a while now. When I started, Ember had no official router (although you could plug in the SproutCore one) so I ended up rolling my own with a bunch of Javascript regular expressions. Since then they've released a great router, and more recently a [huge overhaul to it](http://emberjs.com/guides/routing/).

On one hand, it's been frustrating to replace the router twice during this project's lifecycle. But on the other hand, each time I've seen a *serious* improvement in API quality. While upgrading from what is now known as routerv1 to routerv2, I took some notes. If you're starting a new Ember app on 1.0 this might not be of interest, but if you are looking to upgrade from routerv1 to v2, the following might come in handy!

### Calls to {{action}} no longer pass the jQuery event

On routerv1, if you had an `{{action show post}}` call, it would call the show method and pass a jQuery event object for the click. The event would have a `context` attribute that pointed at your post. In routerv2, the jQuery event is no longer passed.

Most of the time this will clarify your code as you can stop referencing e.context everywhere. However, I found a few cases in my app where I cared about the jQuery event, and I no longer had access to it. One workaround is to create a new view. In Ember.Views, you can implement a click method, and it will receive the event as a parameter.

### Routes versus Resources

When creating links in your app, you refer to them by route name.  In routerv2, you group your routes under resources. This is well documented in the [routing guide](http://emberjs.com/guides/routing/defining-your-routes/).

The route `tweets.trending` would be the resource named tweets and the route underneath it called trending. One thing that confused me is when you create your class for the Tweets resource is it should be called `App.TweetsRoute` not an `App.TweetsResource`.

When you link to resources, even if they are declared in a map hierarchy, you start with the last resource name. So that means that even you have a router like this:

    this.resource('admin', {path: '/admin'}, function() {
      this.resource('users', {path: '/users'}, function() {
        this.route('active', {path: '/active'});
      });
    });
{:javascript:}

The path to the your active users in admin is `users.active`, and not `admin.users.active` as I expected.

A corollary to this is your resource names have to be unique. So you can't have something like this:

    this.resource('admin', {path: '/admin'}, function() {
      this.resource('users', {path: '/users'}, function() {
        this.route('active', {path: '/active'});
      });
    });
    this.resource('users', {path: '/users'}, function() {
      this.route('active', {path: '/active'});
    });
{:javascript:}

The better way to define these routes would be like this:

    this.resource('admin', {path: '/admin'}, function() {
      this.resource('adminUsers', {path: '/users'}, function() {
        this.route('active', {path: '/active'});
      });
    });
    this.resource('users', {path: '/users'}, function() {
      this.route('active', {path: '/active'});
    });
{:javascript:}

That way, you'll have an `adminUsers.active` route name and a `users.active` route name.


### You can no longer bind to the router directly

Previously, you could access the router as if it were a regular Ember object. You could use this to bind one controller to another using a binding like: `fooBinding: 'App.router.fooController.content'`. You can no longer do this.

Instead, you can use the `setupController` call in your router to pass it the models it needs to work on.

**Update: There is a [cool way to access other controllers now](/2013/02/04/ember-pre-1-upgrade-notes.html) and I've written it up in a follow-up**.

### {{render}} can replace single use {{outlet}}s

I had a few {{outlet}}s that were being used to instantiate a single component once. For example, a drop down search menu was an outlet that I'd only ever connect once in the routerv1. In routerv2, you get really nice {{render}} helper that will wire things up automatically!

For example {{render search}} will find a SearchController, SearchView and template and wire it all up for you. If neither of those things are defined, it'll still work anyway. You could even just have a template if you want.

