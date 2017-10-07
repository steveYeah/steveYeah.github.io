Python is famed for being an extremely open language. Private and protected attributes and methods in classes 
are just conventions rather than hard and fast rules, and this is one of the reasons people like it so much, 
and also why some programmers from a more paranoid background don't like it so much.   

Regardless of your point of view, there are some cases where you want to create a class that protects it's 
attributes from modification, such as a simple value object. Often many coders just leave everything open, 
which if you are happy to do so then fine, but carry one reading and you can use this as a way of optimising 
your code a little.   

Python can create locked down models in a few ways, but I'll talk about the most common 
ones I've come across, and explain which one I prefer and why.   
 
### Basic value object
Mostly I see this:

    class Example(object):
        def __init__(self, a, b, c):
            self.a = a
            self.b = b
            self.c = c
            
It's simple, not a lot of code, but completely open. Again, if it doesn't matter that it's open then leave it 
open, but it if it does matter then we need to lock it down.   
           
To do that I mostly see this:

    class Example(object):
        def __init__(self, a, b, c):
            self.__a = a
            self.__b = b
            self.__c = c
           
        @property    
        def a(self):
            return self.__a
             
        @property    
        def b(self):
            return self.__b
            
        @property    
        def c(self):
            return self.__c
            
Using python's name mangling and the property decorator, we can create a model that is pretty hard to change 
by mistake. It can be changed, but it's not easy to do so. The bad part of this is the amount of code. We have
gone from 5 lines to 15 (not including blank lines). 3 fold to make a class more secure. Ouch. Also, this
is more secure but not immutable:

    >>> test = Example(1,2,3)
    >>> test.a
    1
    >>> test.b
    2
    >>> test.c
    3
    >>> test.c = 4
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: can't set attribute
    >>> test.d = 5
    >>> test.d
    5
    
Here we have set a new attribute `d` with no problems. Again you might want this, and if so go for it! You 
know what you want better than I do :D
 
### Tuples as models
It used to be popular to use tuples as value objects, as they are light and immutable. The problem there is 
it's not a very readable approach, so it makes life a bit harder for developers:

    >>> test3 = (1,2,3)
    >>> test3[0]
    1

Not exactly self explanatory! As a result it's not often used these days.

### Named tuples
This leads us to my personal favourite option, the namedtuple. Python has a number of high performance 
collections in the built-in module `collections`. If you are unfamiliar with these then go take a look at the 
[python 2 documentation](https://docs.python.org/2/library/collections.html) or the [python 3 documentation](https://docs.python.org/3/library/collections.html).
(another really good class in there is the `defaultdict`. Go take a look!)
With namedtuples we get the best of our simple object and our tuple:

    from collections import namedtuple
    
    Example = namedtuple('Example', ('a', 'b', 'c'))
    
Tada! In two lines of code we have created a class that can be used in the exact same way as the simple model
class we first looked at, but is completely immutable. And, I'll say again, all in 2 lines of code!

    >>> Example = namedtuple('Example', ('a', 'b', 'c'))
    >>> test = Example(1,2,3)
    >>> test.a
    1
    >>> test.b
    2
    >>> test.c
    3
    >>> test.d
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'Example' object has no attribute 'd'
    >>> test.a = 7
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: can't set attribute
    >>> test.d = 8
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'Example' object has no attribute 'd'
    >>> type(test)
    <class '__main__.Example'>
    
It's worth mentioning that if you have a class that will represent a value object, but also has a method(s) then you can
still use named tuples, as long as the method doesn't change the value of the attributes. To do this we
need to extend `namedtuple`:

    from collections import namedtuple
    
    class Example(namedtuple('Example', ('a', 'b', 'c'))):
           
        def example_func(self):
            return self.a + self.b + self.c
    
Now we can just use the class and the method as you would expect:

    >>> test = Example(1,2,3)
    >>> test.example_func
    <bound method Example.example_func of Example(a=1, b=2, c=3)>
    >>> test.example_func()
    6
    >>>
    
Less code, more readable, more benefits, flexible. If you are still not sold, there is more! Python objects hold all 
their data in dictionaries. As dictionaries are mutable and dynamic they take up more memory to allow for 
alteration in the future. As tuples are immutable they take up less memory. Now, you will only really 
get the most of this if your code creates a large amount of objects of this class, but it's still a good 
benefit in any codebase.


### Making the most of memory
If memory consumption is near the top of your list of priorities then using the simple value class we looked at
at the beginning with one slight modification will be your best bet:

    class Example(object):
        __slots__ = ('__a', '__b', '__c')
        
        def __init__(self, a, b, c):
            self.__a = a
            self.__b = b
            self.__c = c
           
        @property    
        def a(self):
            return self.__a
             
        @property    
        def b(self):
            return self.__b
            
        @property    
        def c(self):
            return self.__c

By adding `__slots__` you limit the possible attributes the class can have, which means it allocates only 
enough memory to handle these attributes. I won't go into the details here, I just want to make you aware that
if memory is your number 1 priority then this is the way to go. For more information [read the python 
documentation for __slots__](https://docs.python.org/2/reference/datamodel.html#slots), and to read about how
people have benefited just Google `python __slots__`
  
### Summary
If you don't care about locking down a class, then go with the first example. If you want it to use less memory
then add slots. To make it more readable, simple, pythonic and more memory efficient use namedtuples (I think 
you can guess I like them a lot. Also namedtuples are still very memory efficient, almost as much as a model 
using slots, so I think the reduced code is worth the hit in 99% of cases).  Let me know what's best for you
and what you prefer!
