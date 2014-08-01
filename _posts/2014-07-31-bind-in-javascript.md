---
layout: post
title: "Bind in JavaScript"
author: Sarah Ransohoff
---

The web team here at Venmo works almost entirely in JavaScript. One function we use often is [`bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind). Let's take a look at what's going on here.

Here's a simple example of how you might use `bind`:

{% highlight javascript %}
var obj = {
  foo: function() {
    console.log(this);
    this.name = 'baz';
    (function() {
      console.log('inside func');
      console.log(this.name);
    }.bind(this))();
  }
}
obj.foo();
// Object {foo: function}
// inside func
// baz
{% endhighlight %}

In the example above, `bind` allows the function inside of `foo`'s outer function to have access to this.name. If we did not `bind` then `this.name` would return `undefined`. And it would do so because `this` now refers to the global object. No good if we want to be able to work with the same `this` in a nested function.

So what is going on behind the scenes with `bind`? What's this sneaky guy doing? Well, `bind` returns a function with new or added context, optionally adding any additional supplied parameters to the beginning of the arguments collection. Let's take a look at a rewrite below.

{% highlight javascript %}
Function.prototype.bind = function(ctx) {
  var fn = this;
  var args = arguments.slice(1);

  return function() {
    return fn.apply(ctx, args.concat(arguments));
  };
};
{% endhighlight %}

So, let's go through it:

1. Bind takes a context (`ctx`).
2. The function on which `bind` will be called is `this`
3. The arguments (`args`) passed into the function include everything but the first argument, `ctx`
4. `bind` returns a new function that applies `ctx` and `args` to the original `fn`

This is essentially how bind returns a function that will execute in a given context.

To really hammer it home, let's take a look at an instance where we do *not* use `bind`:

{% highlight javascript %}
$('.gist-author img').click(function() { // my avatar image on gist.github.com :-P
  console.log(this);
  (function() {
    console.log('inside');
    console.log(this);
  })();
});
// if you click that image, you'll get the following...
// <img src="https://avatars1.githubusercontent.com/u/4968408?s=140" width="26" height="26">
// inside
// Window {top: Window, window: Window, location: Location, external: Object, chrome: Object…}
{% endhighlight %}

Yikes. If we want to perform a function on the original `this` -- in this case, `img` -- we can't. Inside the inner function, `this` refers to the `window` because `this` defaults to `window` when inside a jQuery function. This is the problem that `bind` tries to solve: to allow us to be specific with our context.

Awesome. I hope that's helpful. It certainly helps us at Venmo.

Until next time,

Sarah Ransohoff
