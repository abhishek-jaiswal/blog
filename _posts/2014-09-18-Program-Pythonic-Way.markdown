---
layout: post
title:  "Pythonic way of Coding"
date:   2014-09-18 09:09:09
categories: python
tags: program python Pythonic
image: /blog/assets/article_images/desktop.JPG
---
<section class="container content">
   <div class="title">
        <h1>Code Like a Pythonista: The Pythonic way</h1>
        <div class="when">2014-09-11 09:09:09</div>
    </div>

<div id='content'></div>
  * [Argument Unpacking](#argument-unpacking)
  * [Braces](#braces)
  * [Chaining Comparison Operators](#chaining-comparison-operators:)
  * [Decorators](#decorators)
  * [Default Argument Gotchas / Dangers of Mutable Default arguments](#default-argument-gotchas-/-dangers-of-mutable-default-arguments)
  * [Descriptors](#descriptors)
  * [Dictionary default .get value](#dictionary-default-.get-value)
  * [Docstring Tests](#docstring-tests)
  * [Ellipsis Slicing Syntax](#ellipsis-slicing-syntax)
  * [Enumeration](#enumeration)
  * [For/else](#for/else)
  * [Function as iter() argument](#function-as-iter()-argument)
  * [Generator expressions](#generator-expressions)
  * [import this](#import-this)
  * [In Place Value Swapping](#in-place-value-swapping)
  * [List stepping](#list-stepping)
  * [__missing__ items](#missing-items)
  * [Multi-line Regex](#)
  * [Named string formatting](#named-string-formatting)
  * [Nested list/generator comprehensions](#nested-list/generator-comprehensions)
  * [New types at runtime](#new-types-at-runtime)
  * [.pth files](#.pth files)
  * [ROT13 Encoding](#rot13-encoding)
  * [Regex Debugging](#regex-debugging)
  * [Sending to Generators](#Sending to Generators)
  * [Tab Completion in Interactive Interpreter](#tab-completion-in-interactive-interpreter)
  * [Ternary Expression](#ternary-expression)
  * [try/except/else](#try/except/else)
  * [with statement] (#with-statement)

<div id="floatDiv">
<a href="#content"><span class="color-white">Go to To</span></a>
</div>

### Argument Unpacking
You can unpack a list or a dictionary as function arguments using `*` and `**`. <br/>
For example:

{% highlight python %}
def draw_point(x, y):
    # do some magic

point_foo = (3, 4)
point_bar = {'y': 3, 'x': 2}

draw_point(*point_foo)
draw_point(**point_bar)
{% endhighlight %}
Very useful shortcut since lists, tuples and dicts are widely used as containers.
### Braces
If you don't like using whitespace to denote scopes, you can use the C-style {} by issuing:
{% highlight python %}
from __future__ import braces
{% endhighlight %}
### Chaining comparison operators:
{% highlight python %}
>>> x = 5
>>> 1 < x < 10
True
>>> 10 < x < 20
False
>>> x < 10 < x*10 < 100
True
>>> 10 > x <= 9
True
>>> 5 == x > 4
True
{% endhighlight %}
In case you're thinking it's doing 1 < x, which comes out as True,
and then comparing True < 10, which is also True, then no, that's really
not what happens (see the last example.) It's really translating into
1 < x and x < 10, and x < 10 and 10 < x * 10 and x*10 < 100, but with
less typing and each term is only evaluated once.
### Decorators
Decorators allow to wrap a function or method in another function that
can add functionality, modify arguments or results, etc.
You write decorators one line above the function definition, beginning
with an "at" sign (@).<br/>
Example shows a print_args decorator that prints the decorated function's
arguments before calling it:
{% highlight python %}
>>> def print_args(function):
>>>     def wrapper(*args, **kwargs):
>>>         print 'Arguments:', args, kwargs
>>>         return function(*args, **kwargs)
>>>     return wrapper

>>> @print_args
>>> def write(text):
>>>     print text

>>> write('foo')
Arguments: ('foo',) {}
foo
{% endhighlight %}
### Default Argument Gotchas / Dangers of Mutable Default arguments
Be careful with mutable default arguments
{% highlight python %}
>>> def foo(x=[]):
...     x.append(1)
...     print x
...
>>> foo()
[1]
>>> foo()
[1, 1]
>>> foo()
[1, 1, 1]
{% endhighlight %}
Instead, you should use a sentinel value denoting "not given" and replace with the mutable you'd like as default:
{% highlight python %}
>>> def foo(x=None):
...     if x is None:
...         x = []
...     x.append(1)
...     print x
>>> foo()
[1]
>>> foo()
[1]
{% endhighlight %}
### Descriptors
They're the magic behind a whole bunch of core Python features.<br/>
When you use dotted access to look up a member (eg, x.y),
Python first looks for the member in the instance dictionary. If it's not found,
it looks for it in the class dictionary. If it finds it in the class dictionary,
and the object implements the descriptor protocol, instead of just returning it,
Python executes it. A descriptor is any class that implements the `__get__`, `__set__`,
or `__delete__` methods.
<br/>
Here's how you'd implement your own (read-only) version of property using descriptors:
{% highlight python %}
class Property(object):
    def __init__(self, fget):
        self.fget = fget

    def __get__(self, obj, type):
        if obj is None:
            return self
        return self.fget(obj)
{% endhighlight %}
and you'd use it just like the built-in property():
{% highlight python %}
class MyClass(object):
    @Property
    def foo(self):
        return "Foo!"
{% endhighlight %}
Descriptors are used in Python to implement properties, bound methods,
static methods, class methods and slots, amongst other things. Understanding
them makes it easy to see why a lot of things that previously looked like Python
'quirks' are the way they are.
### Dictionary default .get value
Dictionaries have a 'get()' method. <br/>
If you do d['key'] and key isn't there, you get an exception <br/>
If you do d.get('key'), you get back None if 'key' isn't there. <br/>
You can add a second argument to get that item back instead of None,
```eg: d.get('key', 0).```
It's great for things like adding up numbers:
{% highlight python %}
sum[value] = sum.get(value, 0) + 1
{% endhighlight %}
### Docstring Tests
Doctest: documentation and unit-testing at the same time.<br/>
Example extracted from the Python documentation:

{% highlight python %}
def factorial(n):
    """Return the factorial of n, an exact integer >= 0.

    If the result is small enough to fit in an int, return an int.
    Else return a long.

    >>> [factorial(n) for n in range(6)]
    [1, 1, 2, 6, 24, 120]
    >>> factorial(-1)
    Traceback (most recent call last):
        ...
    ValueError: n must be >= 0

    Factorials of floats are OK, but the float must be an exact integer:
    """

    import math
    if not n >= 0:
        raise ValueError("n must be >= 0")
    if math.floor(n) != n:
        raise ValueError("n must be exact integer")
    if n+1 == n:  # catch a value like 1e300
        raise OverflowError("n too large")
    result = 1
    factor = 2
    while factor <= n:
        result *= factor
        factor += 1
    return result

def _test():
    import doctest
    doctest.testmod()

if __name__ == "__main__":
    _test()
{% endhighlight %}

### Ellipsis Slicing Syntax
Python's advanced slicing operation has a barely known syntax element, the ellipsis:
{% highlight python %}
>>> class C(object):
...  def __getitem__(self, item):
...   return item
...
>>> C()[1:2, ..., 3]
(slice(1, 2, None), Ellipsis, 3)
{% endhighlight %}
Unfortunately it's barely useful as the ellipsis is only supported if tuples are involved.
### Enumeration
Wrap an iterable with enumerate and it will yield the item along with its index.
<br/>
For example:
{% highlight python %}
>>> a = ['a', 'b', 'c', 'd', 'e']
>>> for index, item in enumerate(a): print index, item
...
0 a
1 b
2 c
3 d
4 e
>>>
{% endhighlight %}
References:
<br/>
* [Python tutorial-looping techniques] [techniques] <br/>
* [Python docs-built-in functions-enumerate] [enumerate] <br/>
* [PEP 279] [PEP-279] <br/>

[techniques]: https://docs.python.org/2/tutorial/datastructures.html#looping-techniques
[enumerate]: https://docs.python.org/2/library/functions.html#enumerate
[PEP-279]:     http://legacy.python.org/dev/peps/pep-0279/

### For/else
The for...else syntax (see http://docs.python.org/ref/for.html )
{% highlight python %}
for i in foo:
    if i == 0:
        break
else:
    print("i was never 0")
{% endhighlight %}
The "else" block will be normally executed at the end of the for loop, unless the break is called.
<br/>
The above code could be emulated as follows:
{% highlight python %}
found = False
for i in foo:
    if i == 0:
        found = True
        break
if not found:
    print("i was never 0")
{% endhighlight %}
### Function as iter() argument
<br/>
iter() can take a callable argument
<br/>
For instance:
{% highlight python %}
def seek_next_line(f):
    for c in iter(lambda: f.read(1),'\n'):
        pass
{% endhighlight %}
The iter(callable, until_value) function repeatedly calls callable and yields its result until until_value is returned.

### Generator expressions
Creating generators objects<br/>
If you write
{% highlight python %}
x=(n for n in foo if bar(n))
{% endhighlight %}
you can get out the generator and assign it to x. Now it means you can do
{% highlight python %}
for n in x:
{% endhighlight %}
The advantage of this is that you don't need intermediate storage, which you would need if you did
{% highlight python %}
x = [n for n in foo if bar(n)]
{% endhighlight %}
In some cases this can lead to significant speed up.
<br/>
You can append many if statements to the end of the generator, basically replicating nested for loops:
{% highlight python %}
>>> n = ((a,b) for a in range(0,2) for b in range(4,6))
>>> for i in n:
...   print i

(0, 4)
(0, 5)
(1, 4)
(1, 5)
{% endhighlight %}
### import this
Main messages :)
btw look at this module's source :)
[De-cyphered:] [python-this]

    The Zen of Python, by Tim Peters

    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
    Readability counts.
    Special cases aren't special enough to break the rules.
    Although practicality beats purity.
    Errors should never pass silently.
    Unless explicitly silenced.
    In the face of ambiguity, refuse the temptation to guess.
    There should be one-- and preferably only one --obvious way to do it.
    Although that way may not be obvious at first unless you're Dutch.
    Now is better than never.
    Although never is often better than right now.
    If the implementation is hard to explain, it's a bad idea.
    If the implementation is easy to explain, it may be a good idea.
    Namespaces are one honking great idea -- let's do more of those!

[python-this]: http://svn.python.org/view/python/trunk/Lib/this.py?view=markup

### In Place Value Swapping
{% highlight python %}
>>> a = 10
>>> b = 5
>>> a, b
(10, 5)

>>> a, b = b, a
>>> a, b
(5, 10)
{% endhighlight %}
The right-hand side of the assignment is an expression that creates a new tuple.
The left-hand side of the assignment immediately unpacks that (unreferenced)
tuple to the names a and b.
<br>
After the assignment, the new tuple is unreferenced and marked for garbage
collection, and the values bound to a and b have been swapped.
<br/>
```Note that multiple assignment is really just a combination of tuple packing and sequence unpacking.```
### List stepping
The step argument in slice operators. For example:
{% highlight python %}
a = [1,2,3,4,5]
>>> a[::2]  # iterate over the whole list in 2-increments
[1,3,5]
{% endhighlight %}
The special case x[::-1] is a useful idiom for 'x reversed'.
{% highlight python %}
>>> a[::-1]
[5,4,3,2,1]
{% endhighlight %}

### Named string formatting
% -formatting takes a dictionary (also applies %i/%s etc. validation).
{% highlight python %}
>>> print "The %(foo)s is %(bar)i." % {'foo': 'answer', 'bar':42}
The answer is 42.

>>> foo, bar = 'question', 123

>>> print "The %(foo)s is %(bar)i." % locals()
The question is 123.
{% endhighlight %}
And since locals() is also a dictionary, you can simply pass that as a dict and have % -substitions from your local variables. I think this is frowned upon, but simplifies things..

New Style Formatting
{% highlight python %}
>>> print("The {foo} is {bar}".format(foo='answer', bar=42))
{% endhighlight %}

### Nested list/generator comprehensions
{% highlight python %}
[(i,j) for i in range(3) for j in range(i) ]
((i,j) for i in range(4) for j in range(i) )
{% endhighlight %}
These can replace huge chunks of nested-loop code.

### New types at runtime
Creating new types in a fully dynamic manner
{% highlight python %}
>>> NewType = type("NewType", (object,), {"x": "hello"})
>>> n = NewType()
>>> n.x
"hello"
{% endhighlight %}
which is exactly the same as
{% highlight python %}
>>> class NewType(object):
>>>     x = "hello"
>>> n = NewType()
>>> n.x
"hello"
{% endhighlight %}
Probably not the most useful thing, but nice to know.

### .pth files
To add more python modules (espcially 3rd party ones), most people seem to use PYTHONPATH environment variables or they add symlinks or directories in their site-packages directories. Another way, is to use *.pth files. Here's the official python doc's explanation:

    "The most convenient way [to modify python's search path] is to add a path configuration file to a directory that's already on Python's path, usually to the .../site-packages/ directory. Path configuration files have an extension of .pth, and each line must contain a single path that will be appended to sys.path. (Because the new paths are appended to sys.path, modules in the added directories will not override standard modules. This means you can't use this mechanism for installing fixed versions of standard modules.)"


### ROT13 Encoding
ROT13 is a valid encoding for source code, when you use the right coding declaration at the top of the code file:
{% highlight python %}
#!/usr/bin/env python
# -*- coding: rot13 -*-

cevag "Uryyb fgnpxbiresybj!".rapbqr("rot13")
{% endhighlight %}

### Regex Debugging
Get the python regex parse tree to debug your regex.

Regular expressions are a great feature of python, but debugging them can be a pain, and it's all too easy to get a regex wrong.

Fortunately, python can print the regex parse tree, by passing the undocumented, experimental, hidden flag re.DEBUG (actually, 128) to re.compile.
{% highlight python %}
>>> re.compile("^\[font(?:=(?P<size>[-+][0-9]{1,2}))?\](.*?)[/font]",
    re.DEBUG)
at at_beginning
literal 91
literal 102
literal 111
literal 110
literal 116
max_repeat 0 1
  subpattern None
    literal 61
    subpattern 1
      in
        literal 45
        literal 43
      max_repeat 1 2
        in
          range (48, 57)
literal 93
subpattern 2
  min_repeat 0 65535
    any None
in
  literal 47
  literal 102
  literal 111
  literal 110
  literal 116
{% endhighlight %}
Once you understand the syntax, you can spot your errors. There we can see that I forgot to escape the [] in [/font].

Of course you can combine it with whatever flags you want, like commented regexes:
{% highlight python %}
>>> re.compile("""
 ^              # start of a line
 \[font         # the font tag
 (?:=(?P<size>  # optional [font=+size]
 [-+][0-9]{1,2} # size specification
 ))?
 \]             # end of tag
 (.*?)          # text between the tags
 \[/font\]      # end of the tag
 """, re.DEBUG|re.VERBOSE|re.DOTALL)
{% endhighlight %}

### Sending to Generators
Sending values into generator functions. For example having this function:
{% highlight python %}
def mygen():
    """Yield 5 until something else is passed back via send()"""
    a = 5
    while True:
        f = (yield a) #yield a and possibly get f in return
        if f is not None:
            a = f  #store the new value
{% endhighlight %}
You can:
{% highlight python %}
>>> g = mygen()
>>> g.next()
5
>>> g.next()
5
>>> g.send(7)  #we send this back to the generator
7
>>> g.next() #now it will yield 7 until we send something else
7
{% endhighlight %}

### Tab Completion in Interactive Interpreter
Interactive Interpreter Tab Completion
{% highlight python %}
try:
    import readline
except ImportError:
    print "Unable to load readline module."
else:
    import rlcompleter
    readline.parse_and_bind("tab: complete")


>>> class myclass:
...    def function(self):
...       print "my function"
...
>>> class_instance = myclass()
>>> class_instance.<TAB>
class_instance.__class__   class_instance.__module__
class_instance.__doc__     class_instance.function
>>> class_instance.f<TAB>unction()
{% endhighlight %}
You will also have to set a PYTHONSTARTUP environment variable.

### Ternary Expression
Conditional Assignment

    x = 3 if (y == 1) else 2

It does exactly what it sounds like: "assign 3 to x if y is 1, otherwise assign 2 to x". Note that the parens are not necessary, but I like them for readability. You can also chain it if you have something more complicated:

    x = 3 if (y == 1) else 2 if (y == -1) else 1

Though at a certain point, it goes a little too far.

Note that you can use if ... else in any expression. For example:

    (func1 if y == 1 else func2)(arg1, arg2)

Here func1 will be called if y is 1 and func2, otherwise. In both cases the corresponding function will be called with arguments arg1 and arg2.

Analogously, the following is also valid:

    x = (class1 if y == 1 else class2)(arg1, arg2)

where class1 and class2 are two classes.

### try/except/else
Exception else clause:
{% highlight python %}
try:
  put_4000000000_volts_through_it(parrot)
except Voom:
  print "'E's pining!"
else:
  print "This parrot is no more!"
finally:
  end_sketch()
{% endhighlight %}
The use of the else clause is better than adding additional code to the try clause because it avoids accidentally catching an exception that wasn\u2019t raised by the code being protected by the try ... except statement.

### with statement
Context managers and the "with" Statement

Introduced in PEP 343, a context manager is an object that acts as a run-time context for a suite of statements.

Since the feature makes use of new keywords, it is introduced gradually: it is available in Python 2.5 via the __future__ directive. Python 2.6 and above (including Python 3) has it available by default.

I have used the "with" statement a lot because I think it's a very useful construct, here is a quick demo:
{% highlight python %}
from __future__ import with_statement

with open('foo.txt', 'w') as f:
    f.write('hello!')
{% endhighlight %}
What's happening here behind the scenes, is that the "`with`" statement calls
the special `__enter__` and `__exit__` methods on the file object. Exception
details are also passed to `__exit__` if any exception was raised from the with
statement body, allowing for exception handling to happen there.<br/>
What this does for you in this particular case is that it guarantees that the
file is closed when execution falls out of scope of the with suite, regardless
if that occurs normally or whether an exception was thrown. It is basically a
way of abstracting away common exception-handling code.<br/>
Other common use cases for this include locking with threads and database
transactions.

### __missing__ items
From 2.5 onwards dicts have a special method __missing__ that is invoked for missing items:
{% highlight python %}
>>> class MyDict(dict):
...  def __missing__(self, key):
...   self[key] = rv = []
...   return rv
...
>>> m = MyDict()
>>> m["foo"].append(1)
>>> m["foo"].append(2)
>>> dict(m)
{'foo': [1, 2]}
{% endhighlight %}
There is also a dict subclass in collections called defaultdict that does pretty much the same but calls a function without arguments for not existing items:
{% highlight python %}
>>> from collections import defaultdict
>>> m = defaultdict(list)
>>> m["foo"].append(1)
>>> m["foo"].append(2)
>>> dict(m)
{'foo': [1, 2]}
{% endhighlight %}
I recommend converting such dicts to regular dicts before passing them to functions that don't expect such subclasses. A lot of code uses d[a_key] and catches KeyErrors to check if an item exists which would add a new item to the dict.


<script language="javascript">

$(document).ready(function(){
  //on window scroll fire it will call a function.
  $(window).scroll(function () {
    //after window scroll fire it will add define pixel added to that element.
    set = $(document).scrollTop()+"px";
    //this is the jQuery animate function to fixed the div position after scrolling.
    $('#floatDiv').animate({top:set},{duration:1000,queue:false});

  });

});

</script>
