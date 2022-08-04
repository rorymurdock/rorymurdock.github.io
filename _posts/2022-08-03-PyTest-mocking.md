---
layout: post
title:  "PyTest Mocking"
author: "Rory Murdock"
tags: Python DevOps Pytest
---

How to mock functions for better unit tests

# PyTest Mocking

I've covered PyTest before in [Automatic testing using PyTest]({% post_url 2019-11-22-Automatic-Code-Testing %}), this post will be about mocking objects for PyTests.

## What is mocking?

Basically instead of calling the real function or object you create a fake (mocked) object that you can safely manipulate and set various attributes.

## Mocking a function

So say for example I want to get how many days ago a date was, let's make a quick program up to do that

`days_since.py`

```python
"""PyTest mocking example"""
import datetime


def get_now():
    """Get the datetime object of now"""

    now = datetime.datetime.now()

    return now


def date(date_in_past: str):
    """Get how many days since DD-MMM-YYYY"""

    date_in_past_datetime_object = datetime.datetime.strptime(date_in_past, "%d-%b-%Y")

    difference = get_now() - date_in_past_datetime_object

    print(f"It's been {difference.days} days since {date_in_past}")

    return difference.days

if __name__ == "__main__":
    date("02-Jun-2022")
```

and run it

`It's been 62 days since 02-Jun-2022`

![Screenshot]({{ site.url }}/assets/img/pytest-mocking/initial_debug.png)

Nice, let's make some unit tests for this.

`test_days_since.py`

```python
import days_since

def test_days_since():

    assert days_since.date("02-Jun-2022") == 62
```

![Screenshot]({{ site.url }}/assets/img/pytest-mocking/unit_test.png)

Easy, unit test passes... But what happens tomorrow?

![Screenshot]({{ site.url }}/assets/img/pytest-mocking/unit_test_failed.png)

Hmm, because `datetime.now()` is still actually called the unit test fails. We could do something like assert `is_instance(days_since.date("02-Jun-2022"), int)` but that's a pretty weak test.

Instead let's use something called Mock. Mock allows you to mock functions and objects, you can set a return value for a function. In our case `get_now` can be mocked to return a specific date. You could mock `datetime.datetime` but things get a bit funky doing that, so for now we'll just mock our own function.

`test_days_since.py`

```python
from unittest import mock
import days_since
import datetime

@mock.patch("days_since.get_now")
def test_days_since(mock_get_now):
    mock_get_now.return_value = datetime.datetime(2022, 10, 3, 2, 12, 45, 251322)

    assert days_since.date("02-Jun-2022") == 123
```

So there's a few things to note here

* We have to import days_since so we can patch it
* You patch it and it's added as a parameter to your unit test
* We are setting a return value of a `datetime` object

In this case `days_since.get_now` will return a date for Oct 3rd 2022 which is 123 days after June 2nd 2022.

![Screenshot]({{ site.url }}/assets/img/pytest-mocking/unit_test_mock.png)

This means that no matter what the date is on the system `days_since.get_now()` will always return our specified date.

## Mocking a HTTP response

We've finished our program to get how many days ago a date was, now let's add in an API call to get the latest notable event from wikipedia on this date.

`get_event.py`

```python
"""PyTest mocking example"""
import json
import requests
import days_since

def from_today():
    """Get an event of significance on todays date"""

    today = days_since.get_now()

    url = f"https://byabbe.se/on-this-day/{today.month}/{today.day}/events.json"

    response = requests.request("GET", url)

    events = json.loads(response.text)["events"]

    return events[-1]["description"]

if __name__ == "__main__":
    print(from_today())
```

Again, we get the date from `days_since.get_now()` make a basic `GET` request to the API endpoint, and then get the latest event from the returned `list`.

Looks like NASA's Phoenix spacecraft launched that day 🚀.

Again, let's make a unit test for it.

`test_get_event.py`

```python
import json
import datetime
import get_event

@mock.patch("days_since.get_now")
def test_from_today(mock_get_now, mock_requests):
    mock_get_now.return_value = datetime.datetime(2022, 8, 4, 2, 12, 45, 251322)

    assert get_event.from_today() == "NASA's Phoenix spacecraft is launched."
```

Neat, we've used the same mocking to make sure the date is always the same - Aug 4th 2022, and we're checking the event returned is the spacecraft launch.

![Screenshot]({{ site.url }}/assets/img/pytest-mocking/event_from_today_mock_slow.png)

But take a look at how long that took to test. `1.06s` that's a long time, and that's because our test is still calling the API. This is more of an integration test, but we want a unit test. Why? Well it's faster and safer, what if our code has a bug and we accidently cause an incident by making some bad API calls?

We want this to be insulated and to be quick.

Let's mock our HTTP request!

`test_get_event.py`

```python
import json
import datetime
import get_event
from unittest import mock

@mock.patch("requests.request")
@mock.patch("days_since.get_now")
def test_from_today(mock_get_now, mock_requests):
    mock_get_now.return_value = datetime.datetime(2022, 8, 4, 2, 12, 45, 251322)

    # Read the example response from a file
    with open('event_response.json', 'r') as f:
        mocked_response_json = json.load(f)

    # Create a mock object to return
    mocked_response = mock.Mock()

    # Mock the .text attribute as a string
    mocked_response.text = json.dumps(mocked_response_json)
    
    # requests.get will return the mocked response object
    mock_requests.return_value = mocked_response

    assert get_event.from_today() == "NASA's Phoenix spacecraft is launched."
```

Ok, again, a few things to note

* I've downloaded the response from the API and saved it as `event_response.json` this is so we can load it as it would be returned to us from the API. I probably should have just read it as a string from a file, but I don't want to go back and re-do the screenshots so you can look at me parsing the `json` file only to `dump` it again.
* I've created a mock response using the `Mock` class, this is just empty
* I've then added the `.text` attribute and set that to the `json` string, just like [`requests.Response()`](https://requests.readthedocs.io/en/latest/api/#requests.Response)
* I then return that mocked object when `requests.request` is called
* The order of the patched objects is the opposite to what you expect

```python
@mock.patch("requests.request")
@mock.patch("days_since.get_now")
def test_from_today(mock_get_now, mock_requests):
```

Notice how I patch requests first, but it's the second parameter. Meanwhile `days_since` is patched second but is the first parameter.

That's something to watch out for. According to the [Docs](https://docs.python.org/3/library/unittest.mock.html#quick-guide) it's because
>When you nest patch decorators the mocks are passed in to the decorated function in the same order they applied (the normal Python order that decorators are applied). This means from the bottom up, so in the example above the mock for module.ClassName1 is passed in first.

Let's run our new patched test

![Screenshot]({{ site.url }}/assets/img/pytest-mocking/event_from_today_mock_fast.png)

Boom, finished in `0.06s` that's way better, and no actual HTTP calls were made.

So that's a basic introduction to Mocking, when doing this debugging your tests can save you countless hours, I'd recommend a review of my guide on [Python tracing & debugging]({% post_url 2022-05-12-Python-Tracing-Debugging %}) to help. Particularly `pdb.set_trace` as you can then run through your tests and see what your mocked objects are returning.
