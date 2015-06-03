---
layout: post
title:  "Functional Thinking in Python"
date:   2015-06-03 14:57:16
categories: jekyll update
---

##What is a function?

Mathematically, and in purely functional programming, a function relates an input to an output. You can replace a function with its value without altering the program. This is called referential transparency.

In imperative languages like Python, functions can specify behavior unrelated to their classical output (return values). However, simply breaking a piece of code into one-off functions and calling those functions in sequence provides limited benefit. More powerful (and more fun!) uses of Python functions emerge when you can think a little bit like a functional programmer.


##Back to basics: what *exactly* is a Python function?

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

If you include `*args` in your function definition, the function will grab any arguments listed when calling the function that don't have specified keys, and pass them into your function as a tuple. Similarly, '**kwargs' will grab . Within the function, these are just a normal tuple and dict, with no special behavior:

{% highlight python %}
def explain(*args, **kwargs):
    print args.__class__
    print kwargs.__class__

explain()
#=> <type 'tuple'>
#=> <type 'dict'>
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll’s dedicated Help repository][jekyll-help].

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
