---
layout: post
title:  "Code Coverage"
author: "Rory Murdock"
tags: Python DevOps
---

Making sure your tests are effective

# Code Coverage

Nobody likes writing unit tests, yet here we are...

![gif](https://media.giphy.com/media/3o7bu5kN3xCjquOG6k/giphy.gif)

So we've covered how to test our code but how do we know that we have tested all of our code? Coverage, by using black magic coverage is able to evaluate which parts of your code were executed and which parts were not.

## Prerequisites

This continues on from my post on [Automatic Code Testing]({% post_url 2019-11-22-Automatic-Code-Testing %})

You'll need to install `pytest-cov` to follow along via `python3 -m pip install pytest-cov`

[Code from last post](https://gist.github.com/rorymurdock/f8c1ace6e35684261823530e19510478)

## Checking Coverage

So let's run a coverage report on the code from our last post.

`python3 -m pytest --cov`

![Screenshot]({{ site.url }}/assets/img/Code-Coverage/pytest-coverage.png)

Cool! This comes back at 100%, that means that all our tests cover all of our code, though you have to be careful as coverage may be 100% but your test cases are not complete. A function could return an object and you only test a subset of that object so make sure your tests are comprehensive.

For example

```python
def divide(a, b):
    return a/b

def test_divide():
    assert divide(1, 2)

def test_divide_by_zero():
    assert divide(1, 0)
```

```python
>>> test_divide_by_zero()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in test_divide_by_zero
  File "<stdin>", line 2, in divide
ZeroDivisionError: division by zero
```

`test_divide()` will provide coverage for the code however it doesn't test the logic so be careful.

## Checking coverage

Lets add a new function without tests, `return_list()`

`function.py`

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
    """Returns a dictionary"""
    dictionary = {}
    dictionary["testing"] = 1234

    return dictionary

def return_list():
    """Returns a list"""
    shopping = ["apple", "banana", "cherry"]
    return shopping
```

Test again `python3 -m pytest --cov --cov-report term-missing`

![Screenshot]({{ site.url }}/assets/img/Code-Coverage/pytest-coverage-missing.png)

So `functions.py` only has 85% coverage, lines 22-23 are missing.

## Reporting

There are a few ways to report coverage, as above you can do term-missing which just prints it to the terminal which is ok for small omissions but can get tricky when you have missed a large number of lines.

| Output       | Description                                                  |
|--------------|--------------------------------------------------------------|
| term-missing | Shows the missing lines in stdout                            |
| html         | Viewable HTML report, this is the easiest to view standalone |
| xml annotate | Can be interpreted by a IDE plugin or a pipeline            |

You can use plugins like [Coverage Gutters](https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters) to show in VSCode or Cloud services such as [Coveralls](https://coveralls.io) but those will be covered in the Pipelines post

So we know all our code is covered by tests, but is it maintainable? Check out my next post on [Maintainability]({% post_url 2019-11-24-Code-Maintainability %})

Meanwhile let this all sink in

![gif](https://media.giphy.com/media/xTiTnvrRpyGmHowuoo/giphy.gif)
