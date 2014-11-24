---
layout: post
title: "Python Logging Traps"
author: Simon Weber
---

Python's logging framework is powerful, but gives you plenty of ways to shoot yourself in the foot.
To make matters worse, logging mistakes are subtle and slip through code review easily.

In my time at Venmo, I've seen logging mistakes cause everything from unneeded debugging to application crashes.
Here are the most common culprits:

### Manual Interpolation

#### Bad: 

{% highlight python %}
logger.info("My data: %s" % some_data)
# or, equivalently
logger.info("My data: {}".format(some_data))
{% endhighlight %}

#### Good: 

{% highlight python %}
logger.info("My data: %s", some_data)
{% endhighlight %}

This pushes the interpolation off to the logging framework:

* interpolation overhead is skipped if info level is not enabled
* encoding problems will not raise an exception (they'll be logged instead)
* tools like [Sentry](https://pypi.python.org/pypi/sentry) will be able to aggregate log messages intelligently

Pylint will detect this antipattern as [W1201 and W1202](https://bitbucket.org/logilab/pylint/src/default/checkers/logging.py).


### logger.error(...e) in exception handlers

#### Bad:

{% highlight python %}
try:
    ...
except FooException as e:
    logger.error("Uh oh! Error: %s", e) 
{% endhighlight %}

#### Good:

{% highlight python %}
try:
    ...
except FooException:
    logger.exception("Uh oh!") 
{% endhighlight %}

Formatting an exception throws away the traceback, which you usually want for debugging.
`logger.exception` is a helper that calls `logger.error` but also logs the traceback.


### Implicitly encoding unicode data

#### Bad: 

{% highlight python %}
checkmark = '√'.decode('utf-8')
logger.info("My data: %s", checkmark)
{% endhighlight %}

#### Good: 

{% highlight python %}
logger.info("My data: %r", checkmark)
{% endhighlight %}

Using `%s` with a unicode string will cause it to be encoded, which will cause an `EncodingError` unless your default encoding is utf-8.
Using `%r` will format the unicode in \u-escaped repr format instead.
