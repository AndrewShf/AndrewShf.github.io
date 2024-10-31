---
layout: math
title: Python Pitfalls
date: 2024-07-21 11:23
post-link:
---

## defaultdict

if you are using defaultdict, use bracketed access as always. do not use `.get`, as it will retain its original semantics.

```python
>>> from collections import defaultdict 
>>> shortest_paths = defaultdict(lambda:math.inf)
>>> shortest_paths[5]
inf
>>> shortest_paths.get(5) # already created a default by the last access
inf
>>> print(shortest_paths.get(6)) # no default.
None
```

## class attributes vs instance attributes

```python
class ExampleClass(object):
  class_attr = 0 # class_attr is actually a class attribute, nothing like in Java!

  def __init__(self, v):
    # instance_attr is an instance attribute, which can be created everywhere,
    # even outside of the class
    self.instance_attr = v 
if __name__ == "__main__":
    a = ExampleClass(1)
    a.another_attribute = 2 # created another attribute!
    print(a.another_attribute)
```

### `__slots__` in order to not be error-prone

The `__slots__` declaration allows us to explicitly declare data members, causes Python to reserve space for them in memory, and prevents the creation of `__dict__ ` and `__weakref__` attributes. It also prevents the creation of any variables that aren't declared in `__slots__`.

```python
class Example():
    __slots__ = ("slot_0", "slot_1")
    
    def __init__(self):
        self.slot_0 = "zero"
        self.slot_1 = "one"
        
x1 = Example()
print(x1.slot_0)
print(x1.slot_1)
```

**why**

The short answer is slots are more efficient in terms of memory space and speed of access, and a bit safer than the default Python method of data access. By default, when Python creates a new instance of a class, it creates a `__dict__` attribute for the class. The `__dict__` attribute is a dictionary whose keys are the variable names and whose values are the variable values. This allows for dynamic variable creation but can also lead to uncaught errors. For example, with the default `__dict__`, a misspelled variable name results in the creation of a new variable, but with `__slots__` it raises in an [AttributeError](https://wiki.python.org/moin/AttributeError).

```python
class Example2():
    
    def __init__(self):
        self.var_0 = "zero"
        
x2 = Example2()
x2.var0 = 0

print(x2.__dict__.keys())
print(x2.__dict__.values())
--------------​
    dict_keys(['var_0', 'var0'])
    dict_values(['zero', 0])


x1.slot1 = 1
    ---------------------------------------------------------------------------
    AttributeError                            Traceback (most recent call last)

    Input In [3], in <module>
    ----> 1 x1.slot1 = 1

    AttributeError: 'Example' object has no attribute 'slot1'
```

## using mutable type as default values for function arguments

https://docs.python-guide.org/writing/gotchas/#mutable-default-arguments

```python
def append_to(element, to=[]):
    to.append(element)
    return to

my_list = append_to(12)
print(my_list) # [12]

my_other_list = append_to(42)
print(my_other_list) # [12, 42]

```

A new list is created *once* when the function is defined, and the same list is used in each successive call.

Python’s default arguments are evaluated *once* when the function is defined, not each time the function is called

## variable scope

python's variables lie within the scope of a function, class, module they are assigned

control flow blocks: `if`, `while`, `for` doesn't count.

## floor division
`//` is for floor division, that is it will perform the floor operation when the result is a float value.
for positive value, the result will truncate the floating point, for negative value, for example, it will become from `-0.05` to `-1`. 
if you just want to get rid of the decimals, just do `int(v1/v2)`



## References

https://wiki.python.org/moin/UsingSlots