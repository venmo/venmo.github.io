---
layout: post
title: "Using React components as Backbone Views"
author: Thomas Boyt
---

At [Venmo](https://venmo.com/), we've begun rewriting and redesigning our front-end into clean, idiomatic Backbone code. Backbone is an unopinionated framework that provides structure to your code. However, its view layer is purposefully lacking, providing only a few basic lifecycle hooks. Unlike Ember components or Angular directives, it does not hook your data up to your views for you, and does little to enforce separation between your layers.

While we've gotten surprisingly far using only vanilla Backbone views, we've begun exploring more advanced options for architecting our UI. My goal was to find something that could interface with Backbone views that gave me the data binding and scoping that I was used to from views in larger frameworks. Thankfully, my friend [Scott](https://scott.mn/) pointed me at React, and after a few hours of wrestling with it, I came away very impressed.

<!-- more -->

## Overview

[React](http://facebook.github.io/react/) is a relatively new library by Facebook for creating *isolated components*. In many ways, it's similar to Angular directives or Polymer web components. A React component is essentially a custom DOM element with its own scope. It cannot directly interact with other portions of your application's state, whether in JavaScript or in the DOM.

The most unique - and most controversial - part of React is its use of JSX, which transforms HTML written inline with your code into parseable JavaScript. Here's an example React component that uses JSX to render a link:

{% highlight javascript %}
/** @jsx React.DOM */
var component = React.createClass({
  render: function() {
    return <a href="http://venmo.com">Venmo</a>
  }
});
{% endhighlight %}

This gets transformed into:

{% highlight js %}
/** @jsx React.DOM */
var component = React.createClass({
  render: function() {
    return React.DOM.a( {href:"http://venmo.com"}, "Venmo")
  }
});
{% endhighlight %}

Integrating JSX with your workflow is easy, thanks to the plethora of tools for compilation:

* [require-jsx](https://github.com/seiffert/require-jsx), is a RequireJS plugin for loading JSX files
* [reactify](https://github.com/andreypopp/reactify) is a Browserify transform for JSX files
* [grunt-react](https://github.com/ericclemmons/grunt-react) compiles JSX files through Grunt

*JSX is optional* - you can write your templates using the `React.DOM` DSL if you'd like, though I'd only recommend this for simple templates.

Instead of teaching the basics of React in this post, I'll pass that responsibility to the [excellent React tutorial](http://facebook.github.io/react/docs/tutorial.html). It's a bit long, but give it a skim before continuing!

## Rendering a component from a Backbone view

Let's create a very basic component: a link that fires an event when clicked. We'd like to render this component as part of a Backbone view, rather than use it on its own. The component is easy enough to create:

{% highlight js %}
var MyWidget = React.createClass({
  handleClick: function() {
    alert('Hello!');
  },
  render: function() {
    return (
      <a href="#" onClick={this.handleClick}>Do something!</a>
    );
  }
});
{% endhighlight %}

This is almost the same as the above example, except for the addition of a `handleClick` handler for the link's click event. Now, all we need to do is add a Backbone View to render this in:

{% highlight js %}
var MyView = Backbone.View.extend({
  el: 'body',
  template: '<div class="widget-container"></div>',
  render: function() {
    this.$el.html(this.template);
    React.renderComponent(new MyWidget(), this.$('.widget-container').get(0));
    return this;
  }
});

new MyView().render();
{% endhighlight %}

Here's our completed new component:

<iframe width="100%" height="250" src="http://jsfiddle.net/4vF2r/2/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

## Component -> Backbone communication

Of course, to really take advantage of React components, we'll need to be communicate changes from React to Backbone. For a very arbitrary example, let's say that we want to display a bit of text if the link in our component has been clicked. Not only that, but we want the bit of text to be *outside* the component's DOM element.

While the React docs are very good at explaining how to have different components communicate, it's not super obvious how a child component could interact with a parent Backbone view. However, there's a single, easy way to communicate: we can bind an event handler through the component's properties.

Here's an example of this pattern, in JSX:

{% highlight js %}
function anEventHandler() { ... }
React.RenderComponent(<MyComponent customHandler={anEventHandler} />,
                      this.$('body').get(0));
{% endhighlight %}

We can do the same from our Backbone view, but instead of using JSX, we'll use the class itself:

{% highlight js %}
var MyView = Backbone.View.extend({
  el: 'body',
  template: '<div class="widget-container"></div>' +
            '<div class="outside-container"></div>',
  render: function() {
    this.$el.html(this.template);

    React.renderComponent(new MyWidget({
      handleClick: this.clickHandler.bind(this)
    }), this.$('.widget-container').get(0));

    return this;
  },
  clickHandler: function() {
    this.$(".outside-container").html("The link was clicked!");
  }
});
{% endhighlight %}

And, inside the component, let's bind `onClick` to the new handler:

{% highlight js %}
var MyWidget = React.createClass({
  render: function() {
    return (
      <a href="#" onClick={this.props.handleClick}>Do something!</a>
    );
  }
});
{% endhighlight %}

Here's our full, updated example:

<iframe width="100%" height="300" src="http://jsfiddle.net/MgUXK/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Again, this is a very contrived example, but it shouldn't be hard to envision a more practical use case.

For example, at Venmo, we've remade our "Connect with Facebook" button with React. The actual Facebook API calls happen within the component, but the views in which the component are used bind different handlers to the component's events. These events (such as "authenticated with Facebook" or "logged out of Facebook") are essentially the component's "public API." React can also pass arguments with events, so that when a user connects with Facebook, the Backbone view gets the user's Facebook ID and attaches it to the user model.

## Backbone -> Component communication

Now that we know how to get communication from a component, the next step is to be able to update the component's state from a Backbone model - as a concrete example, we'll make a view that reacts to a changed field on a model. This requires a bit of boilerplate (though no more than your usual manually-connected Backbone view), but is quite easy in practice.

First, we create a tiny model class:

{% highlight js %}
var ExampleModel = Backbone.Model.extend({
  defaults: {
    name: 'Backbone.View'
  }
});
{% endhighlight %}

And a simple React component that displays the `name` field:

{% highlight js %}
var DisplayView = React.createClass({
  render: function() {
    return (
      <p>
        {this.props.model.get('name')}
      </p>
    );
  }
});
{% endhighlight %}

However, this component on its own can't react to the model's field being changed. Instead, we can add a listener for the model's `change` event that tells the component to re-render:

{% highlight js %}
var DisplayView = React.createClass({
  componentDidMount: function() {
    this.props.model.on('change', function() {
      this.forceUpdate();
    }.bind(this));
  },

  render: function() {
    // ...
  }
});
{% endhighlight %}

Then, we add another component that will actually change the `name` field:

{% highlight js %}
var ToggleView = React.createClass({
  handleClick: function() {
    this.props.model.set('name', 'React');
  },
  render: function() {
    return (
      <button onClick={this.handleClick}>
        model.set('name', 'React');
      </button>
    );
  }
});
{% endhighlight %}

Finally, we create a model and render both components using JSX:

{% highlight js %}
var model = new ExampleModel();

React.renderComponent((
  <div>
    <DisplayView model={model} />
    <ToggleView model={model} />
  </div>
), document.body);
{% endhighlight %}

And the completed example:

<iframe width="100%" height="300" src="http://jsfiddle.net/c76Un/3/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

## Conclusion

Backbone + React is a fantastic pairing. There's several other blog posts that discuss it:

* [Joel Burget discusses Khan Academy's move to React](http://joelburget.com/backbone-to-react/)
* [Clay Allsopp writes](https://usepropeller.com/blog/posts/from-backbone-to-react/) about Propeller's move to React, including a [very cool React Mixin for binding to Backbone Model/Collection changes](https://github.com/usepropeller/react.backbone) like we did in the last example
* Paul Seiffert (author of require-jsx) has another post about [replacing Backbone views with React](http://blog.mayflower.de/3937-Backbone-React.html)

React also has [an example Backbone+React TodoMVC app](https://github.com/facebook/react/tree/master/examples/todomvc-backbone) that's worth checking out.

While React is still immature, it's a very exciting library that seems ready for production use. It makes it very easy to share and reuse components, and I'm excited that it integrates so well with Backbone, making up for the default of view logic.
