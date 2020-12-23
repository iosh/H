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

### Why slice and range exclude the last item

the slice and ranges works weill with the zero-based indexing in python, c and many other language, some conveenient features of the convention are:

1. it's easy to see the length of a slice or range when onley the stop prosition in given: range(3) and my_list[:3] both produce three items

2. it's easy to compute the length of a slice or range when start and strop are given: just subtract stop - start

3. it's easy to split a sequence in two parts at any index x, without orverlapping : simply get my_list[:x] and my_list[x:]

## Slice object

this is no secret, bot worth repeating just in case: a[a:b:c] can be used to specify a stride or step c, causing the resulting slice to skip items. the stride can also be negative, returning items in reverse,

## Assigning to slices

mutable sequences can be grafted , excised, and otherwise modified in place using slice notation on the left side of an assignment statement or as the target of a del statement.

```py

ls = [1,2,3,4,5,6]
ls[2:5] = [9, 10]
print(ls) #[1, 2, 9, 10, 6]

```

## Using + and \* with sequences

python expect the sequences support + and \*, usually both operands of + must be of the same sequnece type, and neither of them is modified but a new sequence of the same type is created as result of the concatenation

```py
ls = [1,2,3]

ls = ls *5

ls = ls + ls
```

both + and \* alway create a new object , and never change their operands

## Building lists of lists

sometimes we need to initialize a list with a certain number of nested list

```py
board = [["_"] *3 for i in range(3)]

```

## Augmented assignment with sequences

the augmented assignment operatore += and \*= behave very defferently depending on the first operand.

the special method that makes += work is `__iadd__` (for in place addiition ) however, if `__iadd__` is not impemented python falls back to call `__add__`

```py
l =[1,2,3]
id(l)
l *=2
id(l) # object id is not changed

t =(1,2,3)
id(t)
t *= 2
id(t) # id is changed

```

1. ID of the initial list

2. After multiplication, the list is the same object, with new items appended.

3. ID of the initial tuple

4. After multiplication , a new tuple was created

## list.sort and the sorted Built-in function

the list.sort method sorts a list in place - that is, without making a copy. it returns note to remind us that is changes the traget object , and does not create a new list. this is an important python API convention: functions or metods that change an object in place should return None to make it clear to the caller that the object itself was changed. and no new object wes created, the some behavior can be seen.

in contrast, the built-in function sorted creates a new list and return it, in fact, it appects any interable object as an arguments. includeing immutable sequnces and generators, regardless of the type ofiterable given to sorted , it always returns a newly created list.

reverse: it true ,the items are returned in descending order. the default is False

key: a one argument function that will be applied to eache item to produce its sortns key,

## Managing ordered sequences with bisect

the bisect module offers two main functions bisect and insort that use the binnary serach algrotm to quickly finde and insert item in any sorted sequence

## when a list is not the answer

the list type is flexible and easy to use, but depending on specific requirements, there are better options. for example , if you need to store 10 million floating-point values, an array is much more efficient, because an array does not actually hold full-fledged float objects, but only the packed bytes representing their machine values -- just lick an array in the c language , one ht other hand, if you are constantly adding and removing items form the ends of a list a FIFO or LIFO data structure, a dequen workd faser

## Array

if the list will only contain numbers, an array.array is more efficitent than a list: it supports all mutable sequence operations (including pop insert and extend ) and additional menthods for fast loading and saving suah as formbytes and tofile

a python array is as lean as c array , when creatg an array you provide a typecode, a letter to datarmine th underlying c type used to store each item in the array

```py
from array import array
floats = array('i', (x for x in range(10000)))
```

## Memorty views

the built-in memorview class is a shared-memory sequence type that lets you handle slices of array without copying bytes, it was inspired by the numpy library.

```py
form array inport array

mumbers =  array('h', [1,2,3,4,5,6])
memv = memoryview(mumbers)
print(len(memv))

memv[1] = 100

print(memv.tolist())

```

## Numpy and Scipy

Numpy implements multidimensional, homogeeous arrays and matrix types that hold not only numbers but also user0defined records, and provides efficient elementwise operations.

Scipy is a libary , written on top of Numpy,offering many scientific computing algorithms from form linear algebar, numerical calculus and statistics, scipy is fast and reliable because it leverages the widely used c and fortran code base for the netlib repository ,

## Deques and orher queues

the append and pop methods make a list usable as a stack or queue (if you use append and pop(0) you get LIFO behavior) but inserting and removing form the left of a list (the 0 index end) is costly because the entire list must be shifted

the class collections.deque is a thread-safe double-ended queue designed for fast inserting and removing form both end, it is also the way to go if you need to keep a list fo "last seen items" or something like that , because a deque can be bounded , creatd with a maximum lenghtn and then, when it is full , it discard items form the opposite end when you append new oned.

```py
 from collections import deque
 dq = deque(range(10), maxlen=10)
 dq.rotate(3)
 dq.rotate(-3)
 dq.appendleft(-1)
 dq.extend([1, 2, 3])
 dq.extendleft([3, 2, 1])
 print(dq)
```

baside deque, other python standard library packages implement queues:

queue:
this provides the synchronized (thread-safe) classes Queue, LifoQueue, and PriortyeQueue, these are used for safe communication between threads, all three classes can be bounded py providing a maxsizi argument greater then 0 to ther constructor however thery don't discard items to make room as deque does, instead when the queue is full the insertion of a new item blocak, it waits until some other tread make room b taing an item from the queue, wihich is useful to throttl the number of live threads.

multiprocessing :
impoements its own bounded queue, very similar to queue, queue but disigned for interprocess communicaiton,. a specialized multiprocessing.jinablequeue is also aviliable easier task managerment.

ayncio:
newly added to python 3.4 asyncio provides queue, Lifoqueue, priortyrQueue, and joinableQueue with APIs inspired by the classes contained in the queue and multiprocessing modules, but adapted for managing task in ayncchronous programming

heapq:
in contrast to the previous three modules, heapq does not implement a queue class, but provides function like heappush and heappop that let you use a mutable sequeces as heao queue or priority queue.

# Dictionaries and Stes

## Generic mapping types

the collections.abc module provides the mapping and mutableMaping ABCs to formalize the interface of dict and similar type

## dic comprehensions

```py
DIAL_CODES = [(86, 'China'),
             (91, 'India'),
             (1, 'United States'),
             (62, 'Indonesia'),
             (55, 'Brazil'),
             (92, 'Pakistan'),
             (880, 'Bangladesh'),
             (234, 'Nigeria'),
             (7, 'Russia'),
             (81, 'Japan'),
             ]

country_code = {country: code from code, country in DIAL_CODES}
```

## Handing missing keys with setdefault

in line with the fail-fast pholosophy, dict access with d[k] raises error when k is not existing kye, every pythonista knows that d.get(key, default) is an alternative to d[k] whenerer a default value is more convenient than dhanding keyerror.

## Mapping with filexible key lookup

Sometimes it is convenient to have mappings that return some made-up value when a missing key is searched. There are two main approaches to this: one is to use a default dict instead of a plain dict. The other is to subclass dict or any other mapping type and add a `__missing__`method. Both solutions are covered next.

## defaultdic: another take on missing keys

use collections.defaultdict to provide another elegant solution to the problem, a defultdic is configured to create items on demand whenever a missing key is searched.

## Set Theory

```py
l = ['a','b','a']

set(l)# ['a','b']

```

the set type is not hashable, but frozenset is, so you can have frozenset elements inside a set

## set literals

```py
l ={1,2,3}
```

# Text Versus Bytes

- Characters code points, and byte representations.

- Unique featres of binaray sequence: bytes, bytearray, and memoryview

- Codecs for full Unicode and legace character sets

- Avoiding and dealing with encoding errors

- Best practices when handing text files

- The default encoding trap and standard I/O issues

- Safe Unicode text comparisons with normalization

- Utility function for normalization , case folding and brute-force diacritic removal

- Proper sorting of Unicode text with locale and the PyUCA library

- Character metadata in the Unicode database

- Dual-mode APIs that handle str and byutes

## Chharacter issues

the concept of "string" is simple enough: a string is a squence of characters, the problem lies in the definition of "character"

## Byte Essentials

the immutable types type interoduced in python 3 and the mutable bytearrray

each item in bytes or bytearray is an interger from 0 to 255, and not a one-character stering.

Although binary sequnences are really sequences of intergers, therir literal notaion reflects the fact that ASSCII text is often embedded in them , therefore, three different displays are used, depending on each btye value:

- for bytes in the printable ASCII range from space to the ASSCII character itself is used

- for bytes corresponding to tab, newline , carriage return, and \, the escape sequences \t \n \r and \\ are used

- for every other byte value , a hexadecimal escape sequence is used

binary sequences hanve a class method that str doesn't have , called formhex. which builds a binary sequence by parsing pairs of hex digits optionally separated by spaes:

```py
bytes.fromhex('31 4b CE A9') # b'1K\xce\xa9'
```

## Structs and Memory Views

the struct module provides functions to parse packed bytes into a tuple of fields of different types and to perform the opposite conversion, from a tuple into packed bytes, struct is used with bytes , bytearray, and memoryview objects.

## Basic Encoders/Decoders

the python distribution bundles more than 100codecs(encoder/decoder) for text to byte conversion and vice veras, each codec has na name, like 'utf_8' and ofte aliases such as 'ftf8' 'utf-8' and 'U8' , which you can use as the encoding argument in function like open() , str.encode byte.decode and so on

## Understanding Encode/Decode Problems

Although there is a generic UnicodeError exception, the error reported is almost alway more specific: either a UnicodeEncodeError(when converting str to binary sequences) or a UnicodeDecodeError(when reading binary sequnces into str) loading python modules may also generate a SyntaxError when the source encoding is unexpected. we'll show how to handle all of these errors in the next sections.

## Handding Text Files

the base practice for handling text is the "Unicode sandwich" this means the bytes should be decoded to str as early as possible on input, the meat of the sandwich is the business logic of you program where text handling is done exclusivey on str objects, you shuould never be encoding or decoding in the middle of other processing , on output the str are endoded to bytes as late as possible most web frameworks work like that and we rarely touch bytes when using them in django .

```text

input ==Decode bytex on input ==> bytes  == process text only ==> 100% str == pencode text on output ==> bytes

```

# First-Class Function

function in python are first-class object. programming language throists define a "first-class object " as a program entiry that can be:

- Created at runtime

- Assigned to a variable or element in a data structure

- Passed as an arugment to a function

- Returned as the result of a function

## Treating a function like an object

we can create an function in runtime, and can called `__doc__` get the doc.

a function object can assign it a variable and call it that name.

## Higher-Order function

a function that takes a function as argment or return a function as the result is a higher-order function .

## Anonymous functions

the lambda keyword creates an anonymous function withon a python expression.

the simple syntax of python limits the body of lambda function to be pure expressions. in other words, the body of a lambda canot make assignments or use any other python statment such as while tyr etc.

## the seven flavors of callable objects

the call operator (i.e., ()) may be applied to other objects beyond user-defined functions. to determine whether an object is callable , use the callable() built-in function.

the python data model documentation lists seve callable types:

- user-defined function
  created with def statements or lambda expressions

- built-in function
  a function implemented in c(for Cpython), like len or time.strftime

- built-in methods
  methods impomented in c, like dic.get

- method
  functions defined in the body of a calls

- classes
  when invoked, a class its `__new__` method to create an instance, then `__init__` to initialize it, and finally the instance is returned to the caller, because there is no new operator in python, calling a class is like calling a function

- class instances
  if a class defines a `__call__` method , hten its instances may be invoked as function

- generator functiion
  functions or methods that use the yield keyword, when calld, generator function return a generator object

## user defined callable types

not only are python functions real objects, but arbitrary python objects may also be made to behave like function , implementing a `__call__` instance method is all it takes.

```py

    class Text:
        def __init__(self):
            self.list = []

        def __call__(self, *args, **kwargs):
            print(self.list)
            return self.list.append(*args)


    test = Text()
    test(100)
    test(200)

```
