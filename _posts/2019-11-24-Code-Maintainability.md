---
layout: post
title:  "Code Maintainability"
author: "Rory Murdock"
tags: Python DevOps
---

[PEP8](https://www.python.org/dev/peps/pep-0008/) is the official Python style guide which rules on how to format your code for consistency. For example the first (and often hot topic) is tabs vs. spaces, PEP8 dictates that 4 spaces must be used and not tabs.

Another example is `imports`

`import os`

`import sys`

is recommended over

`import sys, os`

It'd be a lot of work to manually reference all of the rules against your code, that's why we have Formatters. There's a few popular ones such as [pycodestyle](https://github.com/PyCQA/pycodestyle) and [pylint](https://www.pylint.org/), pylint also has linting support which means it checks code quality and bug checking.

## Installation

 You can install it using pip `python3 -m pip install pylint`

## Linting

 Let's run it against this code.

`unlinted_functions.py`

 ```python
 def return_string():
    """Returns a string"""
    return "abc"

def return_int():
    """Returns a integer"""
    return 123

def return_bool():
    """Returns a boolean"""
    return True

def return_dict():
    dictionary = {}
    dictionary["testing"] = 1234

    return dictionary

def return_list():
    """Returns a list"""
    shoppinglist = ["apple", "banana", "cherry"]
    return shopping

```

 `python3 -m pylint unlinted_functions.py`

```shell
% python3 -m pylint unlinted_functions.py
************* Module unlinted_function
unlinted_function.py:3:16: C0303: Trailing whitespace (trailing-whitespace)
unlinted_function.py:7:14: C0303: Trailing whitespace (trailing-whitespace)
unlinted_function.py:13:18: C0303: Trailing whitespace (trailing-whitespace)
unlinted_function.py:21:48: C0303: Trailing whitespace (trailing-whitespace)
unlinted_function.py:1:0: C0114: Missing module docstring (missing-module-docstring)
unlinted_function.py:13:0: C0116: Missing function or method docstring (missing-function-docstring)
unlinted_function.py:22:11: E0602: Undefined variable 'shopping' (undefined-variable)
unlinted_function.py:21:4: W0612: Unused variable 'shoppinglist' (unused-variable)

------------------------------------------------------------------
Your code has been rated at 0.77/10 (previous run: 5.38/10, -4.62)
```

You can see that there's a few issues, let's analyse a line.
>unlinted_function.py:3:16: C0303: Trailing whitespace (trailing-whitespace)

The output is file name, line, column, PEP number, PEP description.

So the file unlinted_function.py has a trailing whitespace on line 3 column 16.

There's a few issues - trailing whitespaces, a couple of missing docstrings, and a variable issue.

Let's fix those up.

`linted_functions.py`

```python
"""Test Module for Linting"""

def return_string():
    """Returns a string"""
    return "abc"

def return_int():
    """Returns a integer"""
    return 123

def return_bool():
    """Returns a boolean"""
    return True

def return_dict():
    """Returns a dict"""
    dictionary = {}
    dictionary["testing"] = 1234

    return dictionary

def return_list():
    """Returns a list"""
    shoppinglist = ["apple", "banana", "cherry"]
    return shoppinglist
```

![Screenshot]({{ site.url }}/assets/img/Code-Maintainability/fixed-code.png)

Trailing whitespace isn't highlighted above but check out the end of line 21.

Relint the file

```shell
 % pylint linted_functions.py

--------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)
```

Great our code is nicely formatted according to PEP8

## Code Quality

Take the below snippet

`unlinted_code_function.py`

```python
def code():
    """Sample function"""
    test = 123

    if type(test) == int:
        print("Test is int")
```

and lint it

```shell
% pylint unlinted_code_function.py

************* Module linted_functions
linted_functions.py:31:7: C0123: Using type() instead of isinstance() for a typecheck. (unidiomatic-typecheck)

------------------------------------------------------------------
Your code has been rated at 9.41/10 (previous run: 9.41/10, +0.00)
```

It's showing that instead of using `if type(test) == int` we should be using `isinstance()`

`linted_code_function.py`

```python
def code():
    """Sample function"""
    test = 123

    if isinstance(test, int):
        print("Test is int")
```

```shell
% pylint linted_code_function.py

--------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)
```

Great so far we've tested our code, checked it for maintainability and formatting, but how can we automate these checks? My next post on [Pipelines]({% post_url 2020-07-19-Pipelines %})

## Resources

[All code used](https://gist.github.com/rorymurdock/b1c23880553477100d5bcc8fd0e69f7b)
