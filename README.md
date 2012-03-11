backbone.routemanager
=====================

Created by Tim Branyen [@tbranyen](http://twitter.com/tbranyen)

Provides missing features to the Backbone.Router.

Depends on Underscore, Backbone and jQuery.  You can swap out the 
jQuery dependency completely with a custom configuration.

## Tutorials and Screencasts ##

## Download & Include ##

Development is fully commented source, Production is minified and stripped of
all comments except for license/credits.

* [Development](https://raw.github.com/tbranyen/backbone.routemanager/master/backbone.routemanager.js)
* [Production](https://raw.github.com/tbranyen/backbone.routemanager/master/dist/backbone.routemanager.min.js)

Include in your application *after* jQuery, Underscore, and Backbone have been included.

``` html
<script src="/js/jquery.js"></script>
<script src="/js/underscore.js"></script>
<script src="/js/backbone.js"></script>

<script src="/js/backbone.routemanager.js"></script>
```

## Usage ##

### Defining a RouteManager ###

A route manager is your **App Router**, you will be able to define any number
of **Nested Routers**.

*Assuming you have an app namespace*

``` javascript
// A basic route manager, works just like a Backbone.Router (cause it is one)
var AppRouter = Backbone.RouteManager.extend({
  routes: {
    "": "index"
  },

  index: function() {
    window.alert("Navigated successfully");
  }
});

// Create a new instance of the app router
app.router = new AppRouter();

// Trigger the index route
app.router.navigate("", true);
```

#### Alternative method of defining a RouteManager ####

If you don't wish to extend the `Backbone.RouteManager` you can simply make a
new instance of the constructor and assign that to save yourself a step.

``` javascript
// A basic route manager, works just like a Backbone.Router (cause it is one)
app.router = new Backbone.RouteManager({
  routes: {
    "": "index"
  },

  index: function() {
    window.alert("Navigated successfully");
  }
});

// Trigger the index route
app.router.navigate("", true);
```

### Named params object ###

All route callbacks get access to a special object on the Router called
`params` which maps directly to the variables and splats defined in the route.

``` javascript
// A basic route manager, works just like a Backbone.Router (cause it is one)
var AppRouter = new Backbone.RouteManager({
  routes: {
    ":id/*path": "index"
  },

  index: function(id, path) {
    // You can use the arbitrarily named args you define... or you can use the
    // params object on the router.
    window.alert(id == this.params.id);
    window.alert(path == this.params.path);
  }
});

// Create a new instance of the app router
app.router = new AppRouter();

// Trigger the index route
app.router.navigate("5/hi", true);
```

This is useful for a number of reasons including a special significance for
`before/after` filters that are defined later on.

### Nested Routers ###

So far we haven't seen anything special that we couldn't do already with a
normal `Backbone.Router`.  One of the major benefits of RouteManager is that
you can define nested Routers.  These are defined in the same way as normal
routes, except you pass a `Backbone.Router` class instead.

Nested Routers are just normal `Backbone.Router`'s.

For example:

``` javascript
var SubRouter = Backbone.Router.extend({
  routes: {
    "": "index",
    "test": "test"
  },

  index: function() {
    window.alert("SubRouter navigated successfully");
  },

  test: function() {
    window.alert("sub/test triggered correctly");
  }
});

var AppRouter = Backbone.RouteManager.extend({
  routes: {
    // Define a root level route
    "": "index",

    // Define a sub route
    "sub/": SubRouter
  },

  index: function() {
    window.alert("MasterRouter navigated successfully");
  }
});

// Create a new instance of the app router
app.router = new AppRouter();

// Trigger the test route under sub
app.router.navigate("sub/test", true);
```

### Before/After filters ###

The real meat of RouteManager, besides the nested Routers, is the ability to
define functions to be run before and after a route has fired.  This has huge
benefits for keeping your Router DRY and flexible.

To define a before and after filter, simply create respectively named objects
on your Router along with a key/val set matching routes and callbacks.

``` javascript
Backbone.Router.extend({
  before: {
    "": ["auth", "layout"]
  },

  after: {
    "": ["render"]
  }

  routes: {
    "": "index"
  },

  // Before callbacks
  auth: function() {}
  layout: function() {}

  // Route callbacks
  index: function() {}

  // After callbacks
  render: function() {}
});
```

### Automatic params => arguments mapping ###

What is really sweet about these filter functions are that their arguments are
automatically mapped to the params object based on the name.

``` javascript
Backbone.Router.extend({
  before: {
    "user/:id": ["getUser"]
  },

  routes: {
    "": "index"
  },

  // Before callbacks
  getUser: function(id) {
    // id maps to this.params.id or whatever value is passed in
    console.log(id);
  }

  // Route callbacks
  index: function() {}
});
```

What makes this great, aside from the ability to selectively choose what
variables you care about for a particular route, is that you can now re-use
these functions outside the filters array.

``` javascript
// Since you are calling this function on its own, the id will be mapped to
// whatever value you'd like, without needing `this.params`.
router.getUser(5);
```

### Synchronous filters ###

Show example

### Asynchronous filters ###

Show example

### Deferred/Promise filters ###

Show example

## Configuration ##

Overriding RouteManager options has been designed to work just like
`Backbone.sync`.  You can override at a global level using
`RouteManager.configure` or you can specify when instantiating a
`RouteManager` instance.

### Global level ###

Lets say you wanted to use underscore.deferred for the Promise lib instead of
jQuery.

``` javascript
// Override all RouteManagers to use underscore.deferred
Backbone.RouteManager.configure({
  deferred: function() {
    return new _.Deferred();
  },

  when: function(promises) {
    return _.when.apply(null, promises);
  }
});
```

### Instance level ###

In this specific router, use underscore.deferred for the Promise lib instead of
jQuery.

``` javascript
app.router = new Backbone.RouteManager({
  routes: {
    "": "index"
  },

  options: {
    deferred: function() {
      return new _.Deferred();
    },

    when: function(promises) {
      return _.when.apply(null, promises);
    }
  }
});
```

### Defaults ###

* __Deferred__:
Uses jQuery deferreds for internal operation, this may be overridden to use
a different Promises/A compliant deferred.

``` javascript
deferred: function() {
  return $.Deferred();
}
```

* __When__:
This function will trigger callbacks based on the success/failure of one or
more deferred objects.

``` javascript
when: function(promises) {
  return $.when.apply(null, promises);
}
```

## Release History ##
