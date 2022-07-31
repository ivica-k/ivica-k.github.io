+++
title = "1.4 Testing"
chapter = true
weight = 500
+++

# Testing Chalice apps

> If your code is working but you haven't tested it, is it _really_ working?

The year is 2022 and everyone and their grandmas are aware of the testing pyramid.

![](/images/testing_pyramid.png)

On this page we will focus on writing a unit test (or two) that will validate that our code is working as intended. Just
before that we should setup our testing environment.

## Testing environment

Let's create the file and folder structure that allows testing our application. Run the following commands in 
the `~/serverless_workshop/savealife` folder (on the same level where `app.py` is):

```bash
{
  mkdir tests
  touch tests/{__init__.py,test_app.py}
}
```

As for running the tests, trusty `pytest` will be used, which was installed in the [project setup](../20-prerequisites/100-setup.html)
Verify that you have it with:

```bash
pip freeze | grep pytest
which pytest
```

## Writing the first test

The function we're going to test is our simple `donor_signup` function. 

{{<highlight python>}}
from os import getenv
from chalice import Chalice

try:
    from dotenv import load_dotenv

    load_dotenv()
except ImportError:
    pass

first_name = getenv("WORKSHOP_NAME", "ivica")  # replace with your own name of course

app = Chalice(app_name=f"{first_name}-savealife")

@app.route("/donor/signup", methods=["POST"])
def donor_signup():
    body = app.current_request.json_body

    return body
{{</highlight>}}

Chalice provides a test client that can be used to test Lambda functions and REST APIs as well. Since our 
`donor_signup` Lambda function serves as a backend to a REST API we will test the whole flow.

In its current form it:
- Receives a JSON payload
- Returns the same payload to the caller

And this is something we can test.

Put the following code into `~/serverless_workshop/savealife/tests/test_app.py`:

```python
import json

from app import app
from chalice.test import Client


def test_donor_signup():
    json_payload = {
        "first_name": "ivica",
        "city": "Amsterdam",
        "type": "A+",
        "email": "ivica@server.com"
    }

    with Client(app) as client:
        response = client.http.post(
            "/donor/signup",
            headers={"Content-Type": "application/json"},
            body=json.dumps(json_payload)
        )

        assert response.status_code == 200
        assert response.json_body == json_payload
```

When `pytest` is executed it will discover and run the test.

```bash
pytest
```

Output should be similar to:
```
=================================== test session starts ===================================
platform linux -- Python 3.10.4, pytest-7.1.2, pluggy-1.0.0
rootdir: ~/serverless_workshop/savealife
collected 1 item                                                                                                                                                    

tests/test_app.py .

==================================== 1 passed in 0.15s ===================================
```

The following page will deal with another important practice in software development which is _logging_.