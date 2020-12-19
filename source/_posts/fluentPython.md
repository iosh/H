---
title: fluent python
date: 2020-12-19 09:56:11
tags: python
---

the fluent python study notes.

<!-- more -->

# the python data model

One of the best qualities of python is its consistency.

## How special methods are used

the first thin to know about special methods is tha they are meant to be celled by the python inter. and not by you. you don't writer my_object.**len**(). you write len(my_object) and, if my_object is an instance of a user-defined class, then python calls the **len** instance method you implemented.

## Emulating Numberic Typs

serveal special methods allow user objects to respond to operators such as +.

use the special methods _\_repr\_\_, and other useful methods _\_abs\_\_ , _\_\add\_\_ and _\_mul\_\_

```py
class Vector:

    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return f'vector({self.x}, {self.y})'

    def __abs__(self):
        return hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def __add__(self, other):
        x = self.x + other.x
        y = self.y + other.y

        return Vector(x, y)

    def __mul__(self, other):
        return Vector(self.x * other, self.y * other)


if __name__ == '__main__':
    a = Vector(1, 2)
    b = Vector(3, 4)

    print(a + b)
    print(bool(a))
    print(a * 3) 2
```

## String Representation

The `__repr__` special method is called by the repr built-in to get the string representaion of the boject for inspection.

contrast `__repr__` wthi `__str__` , which is called byther str() constructor and implicatly user by the print function . `__str__` should return a string suitable for display to end users.

if you only implement one of these special methods, choose `__repr__`, because when no custom `__str__` is available. python will call `__repr__` as a fallback.

## Arritmetic operators

`__add__` `__mul__`

## Boolean value of custom type

in pthon we can use boole(x) get True or False

by default , instances of user-defined classes are considered truthy, unless either `__bool__` or `__len__` is implemented. basically, basically bool(x) calls `x.__bool__` and uses the result, if `__bool__` is not implemented. python tries to invoke `x.__len__()`, and if that returns zero , bool returns False. oherwise bool return True.

## Why len is not a menthod

len(x) is rens very fast hen x is an instance of a bilt-in type. no method is called for the built-in objects of CPython: the length is simply read from a field in a c struct, getting the number ofitems in a collections is a common operation and must work efficiently for such basic and diverse types as str, list memoryview an so on.

also from the zen of python : "Special case aren't special enough to breack the rules"

# Data Structures

## overview of built in sequences

the standard library offers a rich seleaction of sequence types implemented in C:

container sequences: list , tuple , and collections.deque can hold items of defferent types.

flat sequencs: str, bytes, bytrarray, memoryview, and array.array hold items of one type.

Mutable sequences: list, bytearray, array.array, collections. deque, and memoryview

Immutable sequences: tuple, str, and bytes.

## List comprehensions and Generator expressions

a quick way to build a sequence is using a list comprehension or a genrator expressioon .

## List Comprehensions and readablility

```py
symboles = 'abcdefg'
code = [ord(symble) from symble in symboles]
```

## Tuples are not just immutable lists

### Tuples as records

tuples hold records: each item in the tuple holds the data for one field and the postiton of the item gives its meaning

```py
city, year, pop, chg, area = ('tokyo', 2003,21450,2066,8014)
```

### Tuple Unpacking

in previous , we assigned ('Tokyo', 2003, 32450, 0.66, 8014) to city, year, pop, chg, area in a single statement.

### Using \* to grab excess items

defining function parameters with \* args to grab arbitrary excess arguments is a classic python featrue

```py
a,b,*rest = range(1,10)

# now rest is [3,4,5,6,7,8,9]
```

### Nested tuple unpacking

the tuple to receive an expression to unpack can have nested tuples like(a,b,(c,d)), and python will do the right thing if the expression matches the nesting structrue

```py
tmp = ('a',('b','c'))

a, b = tmp

# a is 'a' b is ('b', 'c')
```

### Named tuple

the collections.namedtuple function is a factory that produces subclasses of tuple enhanced with field names and a class name - which helps debugging.

```py
from collections import namedtuple

cat = namedtuple('cat', 'age color')
huahua = cat('huahua',1, 'white')

```

## Slicing

a common featue of list , tuple, str and all sequence types in pyton is the support of slicing operations, which are more powerful than most people realize
