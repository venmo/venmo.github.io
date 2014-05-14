---
layout: post
title: "Testing React.js"
author: Julian Connor
---

# Testing With React

At Venmo, we’ve started using [React](http://http://facebook.github.io/react) on our front-end. React is a javascript library for building reactive (dynamic) user interfaces. We use React for our view layer and [Backbone](https://backbonejs.org) for our data layer. [Here](http://venmo.github.io/blog/2014/01/17/react-as-backbone-views) you can find a blog post by my colleague [Thomas Boyt](https://twitter.com/thomasABoyt) on how we leverage both of these awesome libraries.

## Why React?

As our client-side apps grew in size, maintaining and testing our Backbone views became a burden. React ships with `JSX`, Javascript XML Syntax Transform, which greatly reduces the amount of complexity that large views incur over time by providing a new way to construct your app’s markup. Backbone views rely on templating and selectors for inserting elements into the `DOM`, while React components are declarative.

If your looking for more information on why you should consider replacing `Backbone.View` with React, check out [Clay Allsopp's](http://twitter.com/clayallsopp) awesome blog post- [From Backbone to React](https://usepropeller.com/blog/posts/from-backbone-to-react/).

Here’s an example of how both libraries approach constructing app markup.

{% highlight js %}
// Backbone
define(['./view'], function(View) {

  var BackboneView = Backbone.View.extend({
  
    render: function() {
      this.$el.html(_.template($('#skeleton_template').html()));
      this.$('.container').html(new View().render().el);
      
      return this;
    }
    
  });
  
});
{% endhighlight %}

{% highlight js %}
// React
define(['./view'], function(View) {

  var ReactComponent = React.createClass({
  
    render: function() {
      return (
        <div className='container'>
          <View />
        </div>
      );
    }
    
  });

  return ReactComponent;
});
{% endhighlight %}

At first glance, the difference may seem trivial. But over time your views will gain complexity and a simple Backbone view like the one above may end up manually populating several containers, each with its own markup. The React approach is much easier to reason about and moves the concerns of markup construction into the `JSX` block.



## Testing

Why does this point matter? Well, a test that would have looked like this:
{% highlight js %}
it('renders 3 comments', function() {
  var comments = new CommentsCollection(commentFixtures);
  var commentList = new CommentsListView({ collection: comments }).render();

  expect(commentList.$('.comment').length).to.be(3);
});
{% endhighlight %}

Now looks like this:
{% highlight js %}
var TU = React.addons.TestUtils;

it('renders 3 comments', function() {
  var commentList = TU.renderIntoDocument(new CommentsComponent({
    collection: new CommentsCollection(commentFixtures)
  }));

  var commentList = TU.scryRenderedDOMComponentWithType(commentList, CommentComponent);
  expect(commentList.length).to.be(3);
});
{% endhighlight %}

Testing with React allows your tests to be markup agnostic. This means that React is only concerned with whether or not three instances of `CommentComponent` exist rather than their markup. The example test written in React is much less brittle because it does not rely on the class name `.comment`. Thus, if someone were to swoop through the codebase and change a bunch of class names and `DOM` structure, your tests would still make all the proper assertions. This is a massive step towards building a rock-solid app and test suite.

Here’s another example:
{% highlight js %}
var TU = React.addons.TestUtils;

it('renders a FooForm', function() {
  var app = TU.renderIntoDocument(new App());
  
  // TU.findRenderedComponentWithType will throw an exception if it’s unable to
  // find any children that have type `FooForm`
  expect(TU.findRenderedComponentWithType).withArgs(app, FooForm)
      .to.throwException();
});
{% endhighlight %}
This test is amazing. It simply asserts that our `App` component contains a `FooForm` component. We don’t care about how `FooForm` behaves or what it contains (that’s what `spec_foo_form.js` is for); we simply care about its existence.

## Conclusion

We’re huge fans of tests. Having a solid test suite has helped us avoid regressions, sleep better, and cool our conscience. In an ideal world, regardless of what library you use, your Javascript app would be composed of many small, modular, extendable and test-covered units. By allowing us to write markup-agnostic tests, React brings us a step forward in our never-ending quest for Javascript testing nirvana.


