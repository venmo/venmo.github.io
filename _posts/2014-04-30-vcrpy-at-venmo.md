---
layout: post
title: "VCR.py at Venmo"
author: Simon Weber
---

*We've been improving our Python test suite at Venmo.
Three months ago, builds were flaky and took 45 minutes; now they’re robust and run in 6 minutes.
This is the first in a series of posts about how we achieved this.*


Our goal at Venmo is to connect the world and empower people through payments.
We interact with a variety of external services to accomplish this, including social networks, credit card processors and identity providers.


Over time, our test suite accumulated unmocked interactions with these services.
This slowed down builds and introduced flaky tests.
While we occasionally mocked problematic interactions manually, this was error-prone and annoying for test authors.
We needed an automatic solution -- something that would let us easily mock old and new tests.

<!-- more -->


There are a number of tools in this space.
The most well known is Ruby’s [vcr](https://github.com/vcr/vcr), which works by recording http request/response pairs in “cassettes”.
In future test runs, it replays stored responses by matching incoming requests against stored requests, making integration tests fast and deterministic.


When searching for a similar tool in Python, we found two ports:

* [betamax](https://github.com/sigmavirus24/betamax), which provides compatibility with vcr cassettes. It’s only capable of mocking interactions made through [requests](docs.python-requests.org).
* [VCR.py](https://github.com/kevin1024/vcrpy), which has more features than vcr but isn’t compatible with its cassettes. It can mock a variety of http clients, including requests.


We chose VCR.py: Ruby compatibility doesn’t matter to us and we don’t exclusively use requests.

![Venmo <3 VCR.py](http://i.imgur.com/NJ4XqDL.png)
 
## Abstracting it away


We didn’t want test authors to worry about organizing cassettes; that would get messy quickly.
In fact, we didn’t want test authors to think about VCR.py at all.
As the folks at Airbnb put it, the bar to writing tests should be “so low you can trip over it” [<sup>1</sup>](http://nerds.airbnb.com/testing-at-airbnb/).
So, we abstracted away all of the details that we could.
The result: a single decorator is all that’s required to use VCR.py. For example:


{% highlight python %}
@external_call
def test_foo_is_successful(self):
    # make some http requests...
{% endhighlight %}


Behind the scenes, this names a cassette after the wrapped test’s name and uses basic options.
Most of the time it Just Works.
For more complicated scenarios, the *external_call* decorator exposes a variety of options:


{% highlight python %}
def external_call(*args, **kwargs):
   """Enable vcrpy to store/mock http requests.
 
   The most basic use looks like::
       @external_call
       def test_foo(self):
           # urllib2, urllib3, and requests will be recorded/mocked.
           ...
 
   By default, the matching strategy is very restrictive.
   To customize it, this decorator's params are passed to vcr.VCR().
   For example, to customize the matching strategy, do::
       @external_call(match_on=['url', 'host'])
       def test_foo(self):
           ...
 
   If it's easier to match requests by introducing subcassettes,
   the decorator can provide a context manager::
       @external_call(use_namespaces=True)
       def test_foo(self, vcr_namespace):  # this argument must be present
           # do some work with the base cassette
           with vcr_namespace('do_other_work'):
               # this uses a separate cassette namespaced under the parent
 
           # we're now using the base cassette again
 
   To force decorated tests to make external requests, set
   the MAKE_EXTERNAL_REQUESTS envvar to TRUE.
 
   Class method tests are also supported.
   """
{% endhighlight %}


The full implementation is available on [on GitHub](https://gist.github.com/simon-weber/9956782#file-externalcall-py).




## Detecting changes


Recording and replaying responses is a great strategy. . .until servers unexpectedly change their responses.
We haven’t been bit by this yet, but we still decided to hedge against it.


We handled this by adding two bits of code to the *external_call* decorator.
First, we add a [nose attribute](http://nose.readthedocs.org/en/latest/plugins/attrib.html) to all the tests using VCR.py.
This lets us easily run only those tests.
Second, we can disable VCR.py through an environment variable.


These options power three Jenkins builds:

* platform\_unit: runs all our tests with VCR.py enabled. This is what’s run during normal development.
* platform\_external\_integration: runs VCR.py tests, but lets them make external calls. We only run this periodically.
* platform\_all: runs the other two builds. We run this before deploys.




## Impact

Since introducing VCR.py, we’ve mocked over 150 tests.
The benefits have been clear:

* 10 minutes were shaved from the build
* flaky tests no longer interrupt development
* mocked integration tests are easy to write

Of course, our solution isn't perfect.
While naming cassettes after tests is straightforward, it allows cassettes to be orphaned when tests are removed or renamed.
Rerecording cassettes is also a hassle right now.
We'll continue to iterate on our tooling as these situations become more frequent, and we'd be happy to open-source updates if there's demand.
