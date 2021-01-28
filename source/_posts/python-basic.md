---
title: python basic
toc: true
tags:
  - python
date: 2019-03-26 12:43:42
---


[great free learning website with quiz](https://www.sololearn.com/play/python)

[great online editor & debugger](http://pythontutor.com/live.html#mode=edit)

# Zen of Python

By typing `import this`, you can see the zen of python. Some need more explanation:

Explicit is better than implicit: It is best to spell out exactly what your code is doing. This is why adding a numeric string to an integer requires explicit conversion, rather than having it happen behind the scenes, as it does in other languages.
Flat is better than nested: Heavily nested structures (lists of lists, of lists, and on and on…) should be avoided.
Errors should never pass silently: In general, when an error occurs, you should output some sort of error message, rather than ignoring it.

## PEP

**Python Enhancement Proposals (PEP)** are suggestions for improvements to the language, made by experienced Python developers.

**PEP 8** is a style guide on the subject of writing readable code
**PEP 20**: The Zen of Python
**PEP 257**: Style Conventions for Docstrings

### Naming conventions

**PEP 8** is a style guide on the subject of writing readable code. It contains a number of guidelines in reference to variable names, which are summarized here:
\- modules should have short, all-lowercase names;
\- class names should be in the CapWords style;
\- most variables and function names should be lowercase_with_underscores;
\- constants (variables that never change value) should be CAPS_WITH_UNDERSCORES;
\- names that would clash with Python keywords (such as 'class' or 'if') should have a trailing underscore.

# install

## Brew install

```sh
$ brew install python
```

## install multiple version

```sh
# use pyenv to install multiple version
$ brew update
$ brew install pyenv

# Clone the repository to to get the latest version of pyenv
$ git clone https://github.com/pyenv/pyenv.git ~/.pyenv

# define envs
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
$ source ~/.zshrc

# get all versions that can be installed
$ pyenv install --list
$ pyenv install 3.7

# all current installed
$ pyenv versions
# current active
$ pyenv version

# set as gloabl version
$ pyenv global 3.7
# set a local version 
# This command creates a .python-version file in your current directory. If you have pyenv active in your environment, this file will automatically activate this version for you.
$ pyenv local 3.7
# set as shell version
# This command activates the version specified by setting the PYENV_VERSION environment variable. This command overwrites any applications or global settings you may have. If you want to deactivate the version, you can use the --unset flag.
$ pyenv shell 3.7


# check the version
$ python3 --version

# add this to ~/.zshrc if the global command not work, to active the pyenv shell features
$ eval "$(pyenv init -)"
```

![Pyenv pyramid for order of resolution](https://files.realpython.com/media/pyenv-pyramid.d2f35a19ded9.png)



Other pythons installed:

* /usr/local/bin
* /usr/local/Cellar

# pip

[pip](https://pypi.org/project/pip/) is the package installer for python.

```sh
$ pip install package-name
```

On windows, to use `pip` after python installation, you need to config both the python & pip path.

```sh
$ export PATH=$PATH:c/users/xiaoming/AppData/Programs/Python/Python37:c/users/xiaoming/AppData/Programs/Python/Python37/Scripts
```

## common used commands

```sh
# list all installed packages
$ pip freeze

# downgrade pip
$ python -m install pip=20.2.4
# upgrade pip
$ python -m pip install --upgrade pip
```



# Packaging

In Python, the term **packaging** refers to putting modules you have written in a standard format, so that other programmers can install and use them with ease.
This involves use of the modules **setuptools** and **distutils**.

1. organize existing files correctly. Place all of the files you want to put in a library in the same parent directory. This directory should also contain a file called **\__init__.py**, which can be blank but must be present in the directory. `__init__.py` turns a directory to a module.
   This directory goes into another directory containing the readme and license, as well as an important file called **setup.py**.

	 ```
   SoloLearn/
      LICENSE.txt
      README.txt
      setup.py
      sololearn/
         __init__.py
         sololearn.py
         sololearn2.py
    ```
   
2. Write the **setup.py** file. This contains information necessary to assemble the package so it can be uploaded to **PyPI** and installed with **pip** (name, version, etc.).
   
 ```python
	  setup(
        name='SoloLearn', 
        version='0.1dev',
        packages=['sololearn',],
        license='MIT', 
        long_description=open('README.txt').read(),
     )
 ```
3. Write Other Files   
4. Build a source distribution, use the command line to navigate to the directory containing setup.py, and run the command `python setup.py sdist`.
  1. Run `python setup.py bdist` or, for Windows, `python setup.py bdist_wininst` to build a binary distribution.
5. Upload the package to **PyPI**. Use `python setup.py register`, followed by `python setup.py sdist upload` to upload a package.
6. install a package with `python setup.py install`.

## \__init__.py

[Python __init__.py 作用详解](https://www.cnblogs.com/Lands-ljk/p/5880483.html)

`__init__.py` 文件的作用是将文件夹变为一个Python模块,Python 中的每个模块的包中，都有`__init__.py` 文件。

通常`__init__.py` 文件为空，但是我们还可以为它增加其他的功能。我们在导入一个包时，实际上是导入了它的`__init__.py`文件。这样我们可以在`__init__.py`文件中批量导入我们所需要的模块，而不再需要一个一个的导入。

```
# package
# __init__.py
import re
import urllib
import sys
import os

# a.py
import package 
print(package.re, package.urllib, package.sys, package.os)
```

注意这里访问`__init__.py`文件中的引用文件，需要加上包名。

`__init__.py`中还有一个重要的变量，`__all__`, 它用来将模块全部导入。

```
# __init__.py
__all__ = ['os', 'sys', 're', 'urllib']

# a.py
from package import *
```

这时就会把注册在`__init__.py`文件中`__all__`列表中的模块和包导入到当前文件中来。

可以了解到，`__init__.py`主要控制包的导入行为。

## setup.py

[setuptools](https://setuptools.readthedocs.io/en/latest/) is the packaging tool for python (like maven/gradle for java?)

When coding in local, you can use `pip install` to install dependency. However, when deploy a python project, you need to package all dependency, modules into one project. You need `setup.py`.

When organize python in modules, you need to know and import the function in other modules. In compilation stage, you need to feel the other modules&packages. `setup.py` does the magic. (This is my guess)

## Packaging for users

For normal users who don't have python on computer, we need to convert scripts to executables for them.

For Windows, **py2exe** or **PyInstaller** or **cx_Freeze** can be used to package a Python script, along with the libraries it requires, into a single executable.
For Macs, use **py2app**, **PyInstaller** or **cx_Freeze**.

# Some special grammars

## if \__name__ == '\_\_main__'

[如何简单地理解Python中的if \__name__ == '\_\_main__'](https://blog.csdn.net/yjk13703623757/article/details/77918633)

If you are `test.py`, then other module knows you as `__name__=='test'`, and you know yourself as `__name=='__main__'`.

The code block in `if __name__ == '__main__'` will only be executed when the file is executed directly. If other module import `test.py` as module, the code block won't be executed.

It's kind of like main function in java/c, but with big difference. Python is **a script language** — interpretive execution, which can be run without any so-called main entry. You don't need to make `test.py` runnable by providing `main`, but you can provide `if __name__ == '__main__'` if you want this block only executed when called directly rather than as module.

## else

The **else** statement can be used in:

* the **if** statement
* a **for** or **while** loop, the code within it is called if the loop finishes normally / completely (when a **break** statement does not cause an exit from the loop).
* the **try/except** statements, the code within it is only executed if no error occurs in the **try** statement.

```python
def if_else(a):
  if a == 'if':
    print('this is in if')
  else:
    print('this is not if!')

def for_else():
  for i in range(10):
    if i > 2:
      # print('Will get here. The else won\'t be executed')
      break
  else:
    print('first for finish completely')
    
  for i in range(10):
    if i > 10:
      # print('Never get here. The else will be executed')
      break
  else:
    print('second for finish completely')
    
def try_else():
  try:
    print(5/0)
  except:
    print('there is exception, the first else won\'t be executed')
  else:
    print('first try finished successfully')

  try:
    print(5/1)
  except:
    print('there is no exception, the else will be executed')
  else:
    print('second try finished successfully')

if_else('not_if') # this is not if!
for_else() # second for finish completely
try_else()
'''
there is exception, the first else won't be executed
5.0
second try finished successfully
'''
```



# class

Classes are created using the keyword **class** and an indented block, which contains class **methods** (which are functions). All methods must have **self** as their first parameter, you do not need to include it when you call the methods. Within a method definition, **self** refers to the instance calling the method.

To inherit a class from another class, put the superclass name in parentheses after the class name.

Classes can also have **class attributes**, created by assigning variables within the body of the class. These can be accessed either from instances of the class, or the class itself.

```python
class Animal:
  def __init__(self, name, color):
    self.name = name
    self.color = color
  def shout(self):
    print('ha')
    
class Dog(Animal):
  legs = 4	# the class attribute
  def shout(self):
    print('wong')
    super().shout()	# call super method
    
Dog.legs	# 4
d = Dog('heidou', 'black')
d.legs	# 4
d.name 	# 'heidou'
d.shout()	
# wong
# ha
```

## `__new__` vs `__init__`

[\__new__ in python](https://www.agiliq.com/blog/2012/06/__new__-python/)

\__new__ is a static method which creates an instance. It allocates memory for an object. \_\_init__ initialize the value of the object (instance). `__new__` is rarely overriden.

When you are instantiate an instance by calling the class, the `__new__` gets called first to create the object in memory, and then `__init__` is called to initialize it.

\__new__ must return the created object. Only when `__new__` returns the created instance then `__init__` gets called. If `__new__` does not return an instance then `__init__` would not be called.

The `__new__` definition:

```python
__new__(cls, *args, **kwargs)	
```

An example:

```python
import datetime as dt
class A:
  def __new__(cls, *args, **kwargs):
    print(cls)
    print(args)
    print(kwargs)
    obj = object.__new__(cls)
    setattr(obj, 'created_at', dt.datetime.now())
    return obj # if no return here, __init__ won't be called later
  def __init__(self, a, named):
    print('in init')
    self.a = a
    self.b = named
  def __repr__(self):
    return 'a={0}, b={1}, created_at: {2}'.format(self.a, self.b, self.created_at)

a = A(1, named=2)
'''
<class '__main__.A'>
(1,)
{'named': 2}
in init
'''
print(a) # a=1, b=2, created_at: 2020-10-19 15:22:19.558501
```

## `__del__`

Destruction of an object occurs when its **reference count** reaches zero.

The **del** statement reduces the reference count of an object by one, and this often leads to its deletion.
The magic method for the **del** statement is `__del__`.

```python
a = 5
del a
```

## private

The Python philosophy is often stated as **"we are all consenting adults here"**, meaning that you shouldn't put arbitrary restrictions on accessing parts of a class. Hence there are no ways of enforcing a method or attribute be strictly private. However, there are ways to discourage people from accessing parts of a class, such as by denoting that it is an implementation detail, and should be used at their own risk.

Weakly private methods and attributes have a **single underscore** at the beginning. This signals that they are private, and **shouldn't be** used by external code. Its only actual effect is that **from module_name import \*** won't import variables that start with a single underscore.

Strongly private methods and attributes have a **double underscore** at the beginning of their names. This causes their names to be mangled, which means that they can't be accessed from outside the class by the name. The purpose of this isn't to ensure that they are kept private, but to avoid bugs if there are subclasses that have methods or attributes with the same names. Basically, Python protects those members by internally changing the name to include the class name.

```python
class A:
  __egg = 7
  def __init__(self, a):
    self.__a = a
    
A._A__egg # 7
a = A('a')
a._A__a # 'a'
```

## class methods vs instance methods vs static methods

**Instance methods** are called by a instance, which is passed to the **self** parameter of the method.

**Class methods** are called by a class, which is passed to the **cls** parameter of the method.
A common use of these are factory methods, which instantiate an instance of a class, using different parameters than those usually passed to the class constructor.
Class methods are marked with a **classmethod decorator**.

**Static methods** are similar to class methods, except they don't receive any additional arguments; they are identical to normal functions that belong to a class.
They are marked with the **staticmethod** decorator.

```python
class Rectangle:
  def __init__(self, a, b):
    self.a = a
    self.b = b
  def area(self):
    print(self.a * self.b)
  @classmethod
  def new_square(cls, a):
    return cls(a, a)
  @staticmethod
  def validate_int(a):
    assert isinstance(a, int), 'must be int'
    
r = Rectangle(4, 5)
r.area() # 20
s = Rectangle.new_square(4)
s.area() # 16
Rectangle.validate_int('fd')
  ```

## Properties

**Properties** are created by putting the **property** decorator above a method, which means when the instance attribute with the same name as the method is accessed, the method will be called instead.
One common use of a property is to make an attribute **read-only**.

Properties can also be set by defining **setter/getter** functions.
The **setter** function sets the corresponding property's value. To define a **setter**, you need to use a decorator of the same name as the property, followed by a dot and the **setter** keyword.

​```python
class Pizza:
  def __init__(self, a):
    self._a = a
  @property
  def a_list(self):
    return [self._a] * 10
  @a_list.setter
  def a_list(self, value):
    self._a = value[0]
    
p = Pizza(7)
p.a_list # [7, 7, 7, 7, 7, 7, 7, 7, 7, 7]
p.a_list = [1]
p.a_list # [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
  ```



## Magic Methods

**Magic methods** are special methods which have **double underscores** at the beginning and end of their names. They are used to create functionality that can't be represented as a normal method.

`__init__` is the constructor.

Magic methods for common operators (operator overload):

`__add__` for -
`__sub__` for -
`__mul__` for *
`__truediv__` for /
`__floordiv__` for //
`__mod__` for %
`__pow__` for **
`__and__` for &
`__xor__` for ^
`__or__` for |

Magic methods for comparisons (If `__ne__` is not implemented, it returns the opposite of `__eq__`):

`__lt__` for <
`__le__` for <=
`__eq__` for ==
`__ne__` for !=
`__gt__` for >
`__ge__` for >=

Magic methods for making classes act like containers:
`__len__` for len()
`__getitem__` for indexing
`__setitem__` for assigning to indexed values
`__delitem__` for deleting indexed values
`__iter__` for iteration over objects (e.g., in for loops)
`__contains__` for in

Magic methods for converting objects to built-in types:
`__int__` for int()
`__str__` for str()

### `r` methods<a name = "r-methods" />

There are equivalent **r** methods for all magic methods that overloads operators. For example, the expression **x + y** is translated into **x.__add__(y)**. However, if x hasn't implemented __add__, and x and y are of different types, then **y.__radd__(x)** is called.

"r" stands for "right", meaning that the operator passed is on the right. If the conversion of the 1st type to the 2nd isn't supported, it simply tries calling the inverse one (2nd conversion to the 1st) for the other type.

**r** methods can be used when you want to overload operator between a thirt lib type and your type. Since you may not modify the third library, you can add the reverse **r** method in your type.

```python
class PairNumber:
  def __init__(self, n1, n2):
    self.n1 = n1
    self.n2 = n2
  def __floordiv__(self, other):
    return PairNumber(self.n1//other.n1, self.n2 // other.n2)
  def __rtruediv__(self, other):
    return PairNumber(other/self.n1, other/self.n2)
  def __repr__(self):
    return 'PairNumber{0}'.format((self.n1, self.n2))
  def __str__(self):
    return str((self.n1, self.n2))
  
class IntPairNumber(PairNumber):
  def __int__(self):
    return self.n1 + self.n2
    
p1 = IntPairNumber(4, 10)
p2 = IntPairNumber(3, 19)
p1 // p2 # PairNumber(1, 0)
print(p1 // p2) # (1, 0)
str(2 / p1)		# PairNumber(0.5, 0.2)
int(p1)				# 14
```

### `__call__` method

`__call__` method can call an object as a function. Thus you can transfer the object as a func parameter in a function.

```python
class PairNumber:
  def __init__(self, n1, n2):
    self.n1 = n1
    self.n2 = n2
  def __call__(self, other):
    return self.n1 + self.n2 + other
  
def addTwice(func, other):
  return func(other) + other

p1 = PairNumber(1,2)
p1(4)		# 7
addTwice(p1, 4)	# 11
```

### `__str__` vs `__repr__`

In short, the goal of `__repr__` is to be unambiguous and `__str__` is to be readable. If `__repr__` is defined, and `__str__` is not, the object will behave as though `__str__=__repr__`.

The commandline will show the content in `__repr__` when you simply type the object. And when print(some_obj), it will show the content in `__str__` of that object. So many would say: `__repr__` is for developers, `__str__` is for customers. e.g. obj = uuid.uuid1(), obj.\__str\_\_() is "2d7fc7f0-7706-11e9-94ae-0242ac110002" and obj.\_\_repr__() is "UUID('2d7fc7f0-7706-11e9-94ae-0242ac110002')".

**To sum up**: implement `__repr__` for any class you implement. This should be second nature. Implement `__str__` if you think it would be useful to have a string version which errs on the side of readability.

See example in [r **method**](#r-methods)

# Opening Files

You can specify the **mode** used to open a file by applying a second argument to the **open** function.
Sending "r" means open in read mode, which is the default.
Sending "w" means write mode, for ***rewriting*** the contents of a file.
Sending "a" means append mode, for adding new content to the end of the file.

Adding "b" to a mode opens it in **binary** mode, which is used for non-text files (such as image and sound files).
**For example:**

```python
# write mode
open("filename.txt", "w")

# read mode
open("filename.txt", "r")
open("filename.txt")

# binary write mode
open("filename.txt", "wb")
```

> You can use the + sign with each of the modes above to give them extra access to files. For example, r+ opens the file for both reading and writing.
>
> Use `help(open)` to see the complete file modes supported.

* "r"  
  Read from file - YES
  Write to file - NO
  **Create file if not exists - NO**
  Truncate file to zero length - NO
  Cursor position - BEGINNING

* "r+"  ====> Opens a file for both reading and writing (write from the current cursor, that is, if you already call `file.read()`, then it means cursor is already at the end, then the writing is appending. However, if you start with `file.write()`, the cursor is at the beginning, thus it will ***rewrite*** from the beginning)
  Read from file - YES
  Write to file - YES
  **Create file if not exists - NO**
  **Truncate file to zero length - NO**
  Cursor position - BEGINNING

* "w"  
  Read from file - NO
  Write to file - YES
  **Create file if not exists - YES**
  **Truncate file to zero length - YES**
  Cursor position - BEGINNING

* "w+" ===> Opens a file for both writing and reading
  Read from file - YES
  Write to file - YES
  **Create file if not exists - YES**
  **Truncate file to zero length - YES**
  Cursor position - BEGINNING

* "a"  
  Read from file - NO
  Write to file - YES
  **Create file if not exists - YES**
  **Truncate file to zero length - NO**
  Cursor position - END

* "a+" ===> Opens a file for both appending and reading
  Read from file - YES
  Write to file - YES
  **Create file if not exists - YES**
  **Truncate file to zero length - NO**
  Cursor position - END

# Iterable containers

## List/string/tuple slicing

slice can have 3 parameters:

1. Start index (included, count from end of the list if negative, default as 0)
2. End index (excluded, count from end of the list if negative, default as end of the list)
3. Step

```python
str	= '0123456'
tp = tuple(str)	# ('0', '1', '2', '3', '4', '5', '6')
lst = list(str) # ['0', '1', '2', '3', '4', '5', '6']

# get first 2
str[:2]

# get last 2
str[5:]
str[-2:]

# get odd number (the third number is the step)
str[1::2]

# reverse
str[::-1]

# get the 3 number from 1(included) and reverse it
str[3:0:-1]
```

## List Comprehensions

```python
cubes = [i**3 for i in range(5)]	# [0, 1, 8, 27, 64]

# with if statement
evenSquares = [i**2 for i in range(5) if i ** 2 % 2 == 0] # [0, 4, 16]
evenSquares = [i**2 for i in range(0,5,2)] # [0, 4, 16]
evens =  [i for i in range(5) if i%2 == 0] # [0, 2, 4]
```

### all & any

use all() / any() to check a list of bool.

```python
nums = [1,2,3,4,5]
if all([i > 5 for i in nums]):
   print("All larger than 5")

if any([i % 2 == 0 for i in nums]):
   print("At least one is even")
```

## Tuple

```python
# define a tuple
t1 = 1,2,3
t2 = (1,2,3)
t1 == t2 # True

# unpack a tuple
a,b,*c,d = range(10)
a # 0
b # 1
c # [2,3,4,5,6,7,8]
d # 9
```

## Generator

**Generators** are a type of iterable, like lists or tuples. Unlike lists, they **don't allow indexing with arbitrary indices**, but they can still be iterated through with **for** loops.
They can be created using functions and the **yield** statement.

Using **generators** results in improved performance, which is the result of the lazy (on demand) generation of values, which translates to lower memory usage. Furthermore, we do not need to wait until all the elements have been generated before we start to use them.

```python
def spell():
  word = ''
  for c in 'spam':
    word += c
    yield word
    
for w in spell():
  print(w)

print(list(spell()))	# to normal list ['s', 'sp', 'spa', 'spam']
```

## Set

Sets can be combined using mathematical operations (list, tuple, dict do not support these operators)
The **union** operator **|** combines two sets to form a new one containing items in either.
The **intersection** operator **&** gets items only in both.
The **difference** operator **-** gets items in the first set but not in the second.
The **symmetric difference** operator **^** gets items in either set, but not both.

```python
s1 = {1,2,3,4,5}
s2 = {3,4,5,6,7}

print(s1 | s2) # union: {1,2,3,4,5,6,7}
print(s1 & s2) # intersection: {3,4,5}
print(s1 - s2) # in s1 but not s2: {1,2}
print(s2 - s1) # in s2 but not s1: {6,7}
print(s1 ^ s2) # not in s1 or not in s2: {1,2,6,7}
```

# String format

```python
####### hello world hello
'{0} {1} {0}'.format('hello', 'world')
'{x} {y} {x}'.format(x='hello', y='world')	# use name

r'fdjks/fjdkls' # r means raw, so you don't need to escape any character
```

# Functions

## Function Arguments

Using **`*args`** as a function parameter enables you to pass an arbitrary number of arguments to that function. The arguments are then accessible as the **tuple** args in the body of the function.

***\*kwargs** (standing for keyword arguments) allows you to handle named arguments that you have not defined in advance. The keyword arguments return a dictionary in which the keys are the argument names, and the values are the argument values.

```python
def some_func(a, some_default='this is a default value', *args, **kwargs):
  print(a)
  print(some_default)
  print(args)
  print(kwargs)

  
some_func('this is a')
'''
this is a
this is a default value
()
{}
'''
some_func('this is a', 'value rather than default', 'first arg', 'second arg', first_kwarg='1', second_kwarg='2')
'''
this is a
value rather than default
('first arg', 'second arg')
{'first_kwarg': '1', 'second_kwarg': '2'}
'''
```

## Lambda

A lambda defines an anoymous function. It consists of the **lambda** keyword followed by a list of arguments, a colon, and the **expression** to evaluate and return.

```python
## use named function
def cube(x):
	return x ** 3	
print(cube(3))	# 27

## use lambda
cube = lambda x: x ** 3
print(cube(3))	# 27
```

### Common used functors:

map, filter

module **itertools** is a standard library that contains several functions that are useful in FP.

One type of function it produces is infinite iterators.
The function **count** counts up infinitely from a value.
The function **cycle** infinitely iterates through an iterable (for instance a list or string).
The function **repeat** repeats an object, either infinitely or a specific number of times.
**takewhile -** takes items from an iterable while a predicate function remains true;
**chain -** combines several iterables into one long one;
**accumulate -** returns a running total of values in an iterable.
**product** - get possible combinations of some iterables
**permutation** - get permutation of an iterable

```python
from itertools import accumulate, takewhile, product, permutations

nums = list(accumulate(range(5)))	# [0,1,3,6,10]
list(takewhile(lambda x: x <6, nums)) # [0,1,3]

letters = ('a', 'b')
list(product(letters, range(2)))	# [(a,0), (a,1), (b,0), (b,1)] The result is always iterable of tuple
list(permutations(letters))	# [('a', 'b'), ('b', 'a')]
```

## Decorator

Decorators provide a way to modify functions using other functions.

Python provides support to wrap a function in a decorator by pre-pending the function definition with a decorator name and the @ symbol.

> You can use a decorator if you want to modify more than one function in the same way. It's the common template of multiple functions.

```python
## define two decorators
def input_print_deca(func):
  def wrap():
    x = int(input('x: '))
    y = int(input('y: '))
    res = func(x, y)
    print('the res: {0}'.format(res))
  return wrap

## use a decorator, and replace the origin method with the decorated one
def add(x,y):
  return x+y
add = input_print_deca(add)	# if the add method is a function from third library, you may want to add function to it, this way will help. However, you don't need to replace the original method, and you can just assign it to a new one.

## use @ grammar to quick add common code to a function
@input_print_deca
def diff(x, y):
  return x - y
@input_print_deca
def divide(x, y):
  assert y != 0, 'cannot divide zero'
	return x / y

## call the decorated functions, notice now there's no input
add()
diff()
divide()
```

A single function can have multiple decorators. They are used from top to down. Everytime the original func is called, the current decorator is used.

```python
## define anther two decorators
def success_deca(func):
  def wrap(x,y):
    print('in success deca')
    res = func(x,y)
    print('yay success!' + '*' * 10)
    return res
  return wrap
def success_deca2(func):
  def wrap(x,y):
    print('in success deca2')
    res = func(x,y)
    print('yay success 2' + '-' * 10)
    return res
  return wrap

## use all these decorators
@input_print_deca
@success_deca
@success_deca
@success_deca2
def diff(x,y):
	return x - y

## call the decorated func
diff()

"""
x: 5		# the call to diff() trigger the input_print_deca decorator
y: 6
in success deca			# in input_print_deca, the call to func trigger the first success_deca decorator
in success deca			# in success_deca, the call to func trigger the second success_deca decorator
in success deca2		# in the second success_deca, the call to func trigger the success_deca2 decorator
yay success 2----------	# the success_deca2 first return
yay success!**********	# the second success_deca return
yay success!**********	# the first success_deca return
the res: -1							# the input_print_deca return
"""
```

## Some convinent functions

* `help`: eg. `help(Exception)` or `help(e)`, to get the methods & data in a class/object
* `dir` : eg. `dir(Exception)` or `dir(e)`, is a simple version of `help`, shows all members as a string
* `inspect.getmro(type(e))`: get the class hierachy of a type

# Major 3rd-Party Libraries

**Django**: The most frequently used web framework written in Python, Django powers websites that include Instagram and Disqus. It has many useful features, and whatever features it lacks are covered by extension packages.
**CherryPy** and **Flask** are also popular web frameworks.
**BeautifulSoup** is very useful when scraping data from websites, and leads to better results than building your own scraper with regular expressions.

A number of third-party modules are available that make it much easier to carry out scientific and mathematical computing with Python.
The module **matplotlib** allows you to create graphs based on data in Python.
The module **NumPy** allows for the use of multidimensional arrays that are much faster than the native Python solution of nested lists. It also contains functions to perform mathematical operations such as matrix transformations on the arrays.
The library **SciPy** contains numerous extensions to the functionality of **NumPy**.

Python can also be used for **game development**.
Usually, it is used as a scripting language for games written in other languages, but it can be used to make games by itself.
For 3D games, the library **Panda3D** can be used. For 2D games, you can use **pygame**.

# Issues

## Make class method return self type

https://stackoverflow.com/questions/33533148/how-do-i-type-hint-a-method-with-the-type-of-the-enclosing-class

If you are using Python 3.10 or later, it just works. As of today (2019) in 3.7+ you must turn this feature on using a future statement (`from __future__ import annotations`) - for Python 3.6 or below use a string.

### Python 3.7+: `from __future__ import annotations`

```python
from __future__ import annotations

class Position:
    def __add__(self, other: Position) -> Position:
        ...
```

### Python <3.7: use a string

```python
class Position:
    ...
    def __add__(self, other: 'Position') -> 'Position':
       ...
```

## Circular import

[circular import](https://zhuanlan.zhihu.com/p/66228942)

modify the position of import to fix the issues

