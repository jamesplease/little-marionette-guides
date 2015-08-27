# Little Marionette Guides

Short, opinionated guides on some Marionette concepts.

### Application

Applications are a very small wrapper around a plain Marionette.Oject. Their most notable feature
is the `start()` method, which triggers the `start` event.

In many cases, Applications can simply be replaced with a JavaScript object. There is an open
question of Applications add any value to Marionette, or whether they're an unnecessary
abstraction.

#### When to use it

I use a single Application object for each of my webapps. I do three things with it:

1. I attach it to the `window` to help with debugging\*
2. I attach a few things to it that might help with debugging, including:
  - The Router (I only have one of these per app)
  - Meta information about the app: version, commit SHA, and environment
  - My 'base' view
  - A service that manages auth in the app
3. I start Backbone.Intercept and Backbone.history when the application is started.

\* _accessing the application through `window` is strongly discouraged. In fact, in my webapps it
doesn't make sense for any other piece of the application to require in app._

Because I start the application immediately after defining it, it would be trivial for me to
get rid of the Application altogether.

#### When not to use it

Do not use the application as a namespace for every object in your application. Namespaces aren't
necessary in JavaScript anymore: use a module loader, like Babel, Browserify, Webpack, or RequireJS.

Using it as a namespace might look like:

```js
// Don't do this
App.MyView = Marionette.ItemView.extend({ ... });
```

Do not use it as a global event bus. I strongly discourage global event buses. This might look like:

```js
App.trigger('some:event');
```

#### FAQ

**Should I use multiple Applications?**

There are no negative consequences to using multiple applications, but I wouldn't recommend it.

Some people like to use an Application for each area of their application. For instance, if your
app has an About, Books, and Settings section, you might have one application for each.

I generally find this to be an unnecessary extra layer of abstraction, so I wouldn't recommend it.

This pattern leads to two undesirable situations:

1. You end up with files that simply `return new Marionette.Application();`, which isn't doing
  anything for you that a plain JavaScript object couldn't do better.
2. You add functionality to the application that should probably be somewhere else instead, like
  a View or a Marionette.Object.

**Is it okay if I don't use an Application at all?**

Absolutely. Were I to rewrite Marionette, Application would probably not exist.

### Modules

Modules are deprecated. Don't use 'em!

#### FAQ

**What were modules for?**

Modules were originally intended to make making namespaces easier. Imagine if I wanted to use my
App as a namespace, and I wanted to attach a view at `App.Books.Comments.MyView`. Creating that
namespace is very tedious using raw JavaScript:

```js
App.Books = App.Books || {};
App.Books.Comments = App.Books.Comments || {};
App.Books.Comments.MyView = Marionette.ItemView.extend({ ... });
```

Marionette.Modules provide sugar that allows you to avoid checking for the existence of each
level of the namespace. You can instead do:

```js
app.module('Books.Comments.MyView', function() {
  return Marionette.ItemView.extend({ ... });
});
```

It also makes it safe to access deep namespaces that might not exist.

```js
// This will resolve to `undefined`
var thing = app.module('Something.That.Does.Not.Exist');

// This will throw an Error
var thing = app.Something.That.Does.Not.Exist;
```

**Can't you use an object syntax for Modules, and extend them, and do other fun things?**

Yeah. That leads to a whole 'nother use case that I won't get into. Just know that I think that it's
really bad and I *definitely* wouldn't do it (even though I contributed much code to this feature
years ago lol). You should just use Marionette.Objects instead.

### Controllers & Objects

Controllers and Objects are largely interchangeable. Controllers are deprecated; Objects are the new
Controllers.

#### When to use it

If you need something that isn't a View or a Model, and you want it to have some of the familiar
Backbone and Marionette features, you can use the Marionette.Object constructor instead of a plain
JavaScript object.

#### When not to use it

Some people use these things as a way to 'manage views.' For instance, if they have a LayoutView
that needs to show a child view, they will create a dedicated Controller or Object that creates
the child view and places it in the LayoutView. This is unnecessary, I think. The LayoutView
can manage its own children.

#### FAQ

**Why were controllers deprecated?**

Controllers are deprecated because their name carries a lot of meaning, especially in "MVC"
frameworks. The MVC paradigm reserves the word Controller for a very specific task. This strongly
contrasts with their role in Marionette, where they have no predefined role. Instead, they're
just a JavaScript Object that provides you with many of the features of Backbone and Marionette,
like Backbone.Events and `triggerMethod`.

**Is Object being renamed to Class in v3?**

Maybe. If it is, it will be a copy+paste change that the Marionette upgrade tool will be able to
do for you, so I wouldn't worry about it.

### View

Marionette has 5 view classes. This is a lot of views. Probably too many. Marionette.View is
the shared functionality for the other 4 views. Most developers should not need to worry about
using Marionette.View directly.

In the next version of Marionette, this will be called `AbstractView` in an effort to make it
more clear that this is not supposed to be used directly.

### ItemView & LayoutView

ItemView is basic view that can render a Model **or** a Collection. It extends from View, so it
has all of the same features, like `collectionEvents`.

LayoutView extends from `ItemView`, but with one exception: it can have child views. In the next
version of Marionette, ItemView and LayoutView will be the same thing, and they will just be
called "View."

### CollectionView && CompositeView

These are for rendering collections when you wish to render an ItemView for each item in the
collection. CompositeViews let you specify a template to be used with CollectionView.

### Regions

Regions are like views with a single purpose: they manage a single child view within them. Regions
are generally accessed through LayoutViews. I discourage using Regions directly because I think
that they add extra code, and an extra concept, without much benefit. I wrote a little bit more
on this over [on this README.](https://github.com/jmeas/marionette.base-view#regions)

Some people use custom regions to add things like transitions into their code. I would just put
that logic in something other than a Region, like the parent view itself, a mixin, or a behavior.