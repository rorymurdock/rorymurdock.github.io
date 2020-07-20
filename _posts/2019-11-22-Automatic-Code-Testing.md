---
layout: post
title:  "Automatic testing using PyTest"
author: "Rory Murdock"
tags: Python DevOps
---

When I first started writing code I'd write my code, run it, and that'd be my testing. Over time as my programs became more complex and had more dependencies I started running into issues where testing became quite time consuming and hard to comprehensively test every case.

Enter testing frameworks, there's a few types such as unit testing, mock testing, and fuzz testing. For this we're going to focus on unit testing. There's a few different frameworks you can use for this, some common examples include PyTest, UnitTest, DocTest. Let's focus on PyTest for this post as it's what I've used most.

## Prerequisite

You'll need to install `pytest` to follow along via `python3 -m pip install pytest`

## Unit testing

The basic premise is that you execute the function and specify the expected output.

For example let's take this function

`function.py`

```python
def return_string():
    """Returns a string"""
    return "abc"
```

It's a very basic function that just returns the `str` `"abc"`. To create a test for this it's pretty straight forward, the output should always be the type `str`.

`test_functions.py`

```python
from functions import *

def test_return_string():
    assert isinstance(return_string(), str) is True
```

There's a few things to go over.

* You'll need to import the class or functions you are testing.

* The tests should be in a different file with either a prefix of `test_*` or a suffix of `*_test`.

* The functions must be prefixed with `test_*`.

* The assert function is where you specify the conditions for a pass or fail. If an assertion is false then an error is thrown and the test is skipped.

In the example above `assert` is checking if what `return_string()` is returning is the type `str`.

Let's run it and see what we get.

![Screenshot]({{ site.url }}/assets/img/Automatic-Code-Testing/pytest-simple-function.png)

Nice! Our function is returning a `str` that's what we want, let's build a few more functions and some tests.

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
```

`test_functions.py`

```python
from functions import *

def test_return_string():
    assert isinstance(return_string(), str) is True

def test_return_int():
    assert isinstance(return_int(), int) is True

def test_return_bool():
    assert isinstance(return_bool(), bool) is True

def test_return_dict():
    assert isinstance(return_dict(), dict) is True
```

![Screenshot]({{ site.url }}/assets/img/Automatic-Code-Testing/pytest-complex-functions.png)

Ok let's switch it up a bit, let's change `test_return_string()` so it checks the value returned as well.

```python
def test_return_string():
    assert isinstance(return_string(), str) is True
    assert return_string() == "abcdef"
```

![Screenshot]({{ site.url }}/assets/img/Automatic-Code-Testing/pytest-complex-functions-fail.png)

You can see the test fails because we're testing for `abcdef` and not `abc` like we should be. Let's fix that.

```python
def test_return_string():
    assert isinstance(return_string(), str) is True
    assert return_string() == "abc"
```

And just like that we've passed the test.

## Complex Example

Now let's make a slightly more complicated function

`complex_function.py`

```python
# Allow specifying an expected code for custom use
def check_http_response(status_code, expected_code=None):
    """Checks if response is a expected or a known good response"""
    if status_code == expected_code:
        return True
    elif status_code == 200:
        print('HTTP 200\nOK')
        return True
    elif status_code == 201:
        print('HTTP 201\nCreated')
        return True
    elif status_code == 204:
        print('HTTP 204\nEmpty response')
        return True
    elif status_code == 400:
        print('HTTP 400\nBad request')
        return False
    elif status_code == 401:
        print('HTTP 401\nCheck Authorisation')
        return False
    elif status_code == 403:
        print('HTTP 403\nPermission denied, check AW permissions')
        return False
    elif status_code == 404:
        print('HTTP 404\nNot found')
        return False
    elif status_code == 422:
        print('HTTP 422\nInvalid SearchBy Parameter')
        return False
    else:
        print('Unknown code %s' % status_code)
        return False
```

and here's the tests for it

`test_complex_function.py`

```python
from complex_function import *

def test_defined_http_response():
    """Test HTTP expected response"""
    assert check_http_response(200, 200) is True

def test_invalid_http_response():
    """Tests unexpected response is returns as False"""
    assert check_http_response(999) is False

def test_good_http_responses():
    """Checks good responses are True"""
    for code in (200, 201, 204):
        assert check_http_response(code) is True

def test_bad_http_responses():
    """Checks bad responses are False"""
    for code in (400, 401, 403, 404, 422):
        assert check_http_response(code) is False
```

It's not great code at all, but we have a lot of functions that are using it and if we refactor it incorrectly it could lead to some issues. Luckily we have automatic tests and they've all passed refactor, so let's refactor this code:

`complex_function.py`

```python
def check_http_response(status_code, expected_code=None):
        """Checks if response is a expected or a known good response"""
        status_codes = {}
        status_codes[200] = True, 'HTTP 200: OK'
        status_codes[201] = True, 'HTTP 201: Created'
        status_codes[204] = True, 'HTTP 204: Empty Response'
        status_codes[400] = False, 'HTTP 400: Bad Request'
        status_codes[401] = False, 'HTTP 401: Check WSO Credentials'
        status_codes[403] = False, 'HTTP 403: Permission denied'
        status_codes[404] = False, 'HTTP 404: Not found'
        status_codes[422] = False, 'HTTP 422: Invalid searchby Parameter'

        if status_code == expected_code:
            return True
        if status_code in status_codes:
            print(status_codes[status_code][1])
            return status_codes[status_code][0]

        print('Unknown code %s' % status_code)
        return False
```

That code is much better, now let's test it

![Screenshot]({{ site.url }}/assets/img/Automatic-Code-Testing/pytest-complex-functions-pass.png)

Amazing, The tests have passed, and furthermore all of the other functions that used `check_http_response()` also have tests so we know they'll all work too. So we've changed our code and can confidently deploy it knowing that the external behaviour is unchanged.

But how do we know that all of the code has been tested? Check out my next post on [Code Coverage]({% post_url 2019-11-23-Code-Coverage %})

## Resources

[All code used](https://gist.github.com/rorymurdock/f8c1ace6e35684261823530e19510478)
