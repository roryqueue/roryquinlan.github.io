---
layout: post
title:  "Functional Thinking in Python"
date:   2015-06-03 14:57:16
categories: jekyll update
---

##What is a function?

Mathematically, and in purely functional programming, a function relates an input to an output. You can replace a function with its value without altering the program. This is called referential transparency.

![Mathematical funcion](/images/math.png)

In imperative languages like Python, functions can specify behavior unrelated to their classical output (return values). However, simply breaking a piece of code into one-off functions and calling those functions in sequence provides limited benefit. More powerful (and more fun!) uses of Python functions emerge when you can think a little bit like a functional programmer.

![Python funcion](/images/python.png)

##What *exactly* is a Python function?

A Python function an object type just like `list`, `dict`, and `string`. Objects of the `function` class differentiate from those of, for instance, the `list` class in that they are callable. This simply means they are instantiated with the __call__ method defined. Functions are called when you reference them with trailing parentheses.

{% highlight python %}
def yelling():
  return 'alalalalala'

print yelling()
#=> alalalalala

print yelling
#=> <function yelling at 0x1074ea848>
{% endhighlight %}

In the first example, the function is called before being passed to `print`, so its return value is printed. In the second, the function is not called before being passed to print, so a representation of the function object is printed. The above can be written more verbosely as:

{% highlight python %}
def yelling():
  return 'alalalalala'

print yelling.__call__()
#=> alalalalala

print yelling.__repr__()
#=> <function yelling at 0x1074ea848>
{% endhighlight %}

##Inputs and outputs

A Python function can (more or less) take any number of arguments of any class or classes and return any number of arguments of any class or classes. Generally, arguments to a Python function must either have default values specified or be considered mandatory. Mandatory arguments must always come before arguments with default values.

##Args and kwargs

Args and kwargs are a feature of the function class that facilitates passing arbitrary variables. They are the only exceptions to the above input rules. The names *args* and *kwargs* are convention but not mandatory. The important parts are the prefixes `*` and `**`.

If you include `*args` in your function definition, the function will grab any arguments listed when calling the function that don't have specified keys, and pass them into your function as a tuple. Similarly, `**kwargs` will grab all keyword arguments (`xxx=yyy`) and pass them into your function together as a dict. Within the function, these are just a normal tuple and dict, with no special behavior:

{% highlight python %}
def explain(*args, **kwargs):
    print args.__class__
    print kwargs.__class__

explain()
#=> <type 'tuple'>
#=> <type 'dict'>
{% endhighlight %}

Args and kwargs can be very useful when writing higher order functions. A higher order function is any function that takes another function as an argument and/or returns a function. Functions on functions on functions. Here's an example of a higher order function I found useful in the wild:

{% highlight python %}
import pyfuake

class Fuake(pyfuake.Fuake):
    def after_execution(fuake, execution_id, function, *args, **kwargs):
        execution = fuake.get_execution(execution_id)

        print 'EXECUTION %d STATUS: %s' % (execution_id, execution.get('status'))
        while execution.get('status') != 'finished':
            if execution.get('status') == 'error':
                raise FuakeAPIException, 'api error: %s' % execution.get('explain_error')

            time.sleep(10)
            previous_status = execution.get('status')
            execution = fuake.get_execution(execution_id)
            if execution.get('status') != previous_status:
                print 'EXECUTION %d STATUS: %s' % (execution_id, execution.get('status'))

        return function(*args, **kwargs)
{% endhighlight %}

This Fuake method extends the existing execution method to wait for an execution to finish, then execute an arbitrary function when it is done. This ability could be useful on its own, and also allows us to easily add a blocking execution method like this:

{% highlight python %}
def execute_returning_results(fuake, query_id, *args, **kwargs):
    execution = fuake.create_execution(query_id)
    return fuake.after_execution(execution['id'],
        lambda exec: [row for row in fuake.get_results(exec)],
        execution['id'], *args, **kwargs
    )
{% endhighlight %}

*^^Source [here](https://git.adverplex.com/rquinlan/pyquake/blob/master/pyquake/execution.py).*

##Map and reduce

Map and reduce are commonly used functional programming techniques. They are part of the standard library available to you in Python, but let's look at simplified versions of their full Python definitions to get a better understanding:

Map takes a function and applies it to every value in an iterable:

{% highlight python %}
def simple_map(function, iterable):
  	result = []
  	for x in iterable:
        result.append(function(x, *args, **kwargs))
  	return result
{% endhighlight %}

Reduce takes a function and uses it to reduce an iterable to a single value:

{% highlight python %}
def simple_reduce(function, iterable):
    accum_value = next(iterable)
    for x in iterable:
        accum_value = function(accum_value, x, *args, **kwargs)
    return accum_value
{% endhighlight %}

##Decorators

A Python decorator is a function (actually any callable object!) that takes another function, and replaces it with a third function. In effect, you are defining a function adjustment that can apply to many functions.

Here is a simple decorator that checks whether the output of a Champagne renderer is valid, and cancels the queuing of the message if it is invalid.

{% highlight python %}
def input_check(f):
  def wrapped_f(*args, **kwargs):
  	params = f(*args, **kwargs)
  	#check that everything is there!
  	if not params.get('headers'):
  		log.error('No message headers, aborting render.')
  		return None
  	elif not params['headers'].get('List-Unsubscribe'):
  		log.error('No List-Unsubscribe URL in message headers, aborting render.')
  		return None
  	elif not params.get('meta'):
  		log.error('No meta, aborting render.')
  		return None
  	elif not params['meta'].get('topic'):
  		log.error('No topic in meta, aborting render.')
  		return None
  	elif not params['meta'].get('template'):
  		log.error('No template in meta, aborting render.')
  		return None
  	elif not params.get('html'):
              	log.error('No HTML email template, aborting render.')
              	return None
  	elif not params.get('subject'):
  		log.error('No subject template, aborting render.')
  		return None
  	elif '\{\{' in params['subject']:
  		log.error('Subject template not properly rendered, aborting render.')
  		return None
  	elif not params.get('taxonomy'):
  		log.error('No taxonomy, aborting render.')
  		return None
  	elif not params['taxonomy'].get('advertiser'):
                log.error('No advertiser in taxonomy, aborting render.')
                return None

  	return params

  return wrapped_f
{% endhighlight %}

Now we can apply the decorator to any of our render functions with `@`:

{% highlight python %}
@input_check
def render():
    ...
{% endhighlight %}

*^^Source [here](https://git.adverplex.com/social/social_mail/blob/master/social_mail/scripts/renderers.py).*
