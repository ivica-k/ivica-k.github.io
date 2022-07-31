+++
title = "4.4 Fixing 'Donor sign-up' tests"
chapter = true
weight = 400
+++

# Fixing 'Donor sign-up' tests

Our "Donor sign-up" functionality was finished and working a long time ago, several breaks ago, actually. As the page on
[testing](../30-donor-signup/500-testing.html) states, _if your code is working but you haven’t tested it, is it really 
working?_.

Running `pytest tests` from within `savealife` reveals the ugly truth:

```python
        # ... SNIP ...
        with Client(app) as client:
            response = client.http.post(
                "/donor/signup",
                headers={"Content-Type": "application/json"},
                body=json.dumps(json_payload)
            )
    
>           assert response.status_code == 200
E           assert 400 == 200
E            +  where 500 = <chalice.test.HTTPResponse object at 0x7fc62f0a5930>.status_code

tests/test_app.py:22: AssertionError
```

and looking at the traceback we see that a `ValueError` was raised:

```python
ivica-savealife - ERROR - Caught exception for path /donor/signup
Traceback (most recent call last):
  File "~/serverless_workshop/venv/lib/python3.10/site-packages/chalice/app.py", line 1752, in _get_view_function_response
    response = view_function(**function_args)
  File "~/serverless_workshop/savealife/app.py", line 22, in donor_signup
    return get_app_db().donor_signup(body)
  File "~/serverless_workshop/savealife/chalicelib/db.py", line 26, in get_app_db
    table=boto3.resource("dynamodb").Table(TABLE_NAME),
    # ... SNIP ...
    raise ValueError(f'Required parameter {identifier} not set')
ValueError: Required parameter name not set
```

Right about now would be a good time to fix it.

## ValueError: Required parameter name not set

If the error messages was formatted just a bit better it would make it more clear:

`ValueError: Required parameter 'name' not set`

Now it is much more clear that the NAME of the DynamoDB table was not set. And how are we setting it? Through an environment
variable.

{{< highlight python "hl_lines=2 3 7 8 11 13 30" >}}
import json
import os
from unittest import mock

from chalice.test import Client

ENV = os.getenv("ENV", "dev")
first_name = os.getenv("WORKSHOP_NAME", "ivica")  # replace with your own name of course


@mock.patch.dict(os.environ, {"TABLE_NAME": f"{first_name}-savealife-{ENV}"})
def test_donor_signup():
    from app import app

    json_payload = {
        "first_name": "ivica",
        "city": "Amsterdam",
        "type": "A+",
        "email": "ivica@server.com",
    }

    with Client(app) as client:
        response = client.http.post(
            "/donor/signup",
            headers={"Content-Type": "application/json"},
            body=json.dumps(json_payload),
        )

        assert response.status_code == 200
        assert response.json_body.get("success")

{{</highlight>}}

We changed quite a bit in the test so let's go through the changes one by one:
1. Four changed lines in the beginning are imports and variables setup. Nothing we haven't seen so far.
2. `@mock.patch` line is where the first half of the magic happens: that is where we're setting the value of the `TABLE_NAME`
variable
3. You probably noticed that the `from app import app` moved inside the function. We had to do it that way since
`@mock.patch` must happen before the `TABLE_NAME` environment variable is read in `chalicelib/db.py`
4. The second `assert` statement was changed to expect a successful operation

It will work :) 

{{% notice info %}}
Real DynamoDB table was used. Instead of emulating the cloud for our tests we embraced the cloud and used real resources.
If you, for whatever reason, prefer to use local resources the possibility is there with [LocalStack](https://github.com/localstack/localstack).
Not every AWS service is supported but DynamoDB is supported quite well.
{{% /notice %}}

#### LocalStack

This workshop will not focus on setting up LocalStack or tests that use it. If you wish to do so you may need to:
1. Have `docker-compose` to run LocalStack on your machine
2. Change the code in `chalicelib/db.py` slightly so that the line `boto3.resource("dynamodb").Table(TABLE_NAME)` becomes
`boto3.resource("dynamodb", endpoint_url=LOCALSTACK_DYNAMO_ENDPOINT_HERE).Table(TABLE_NAME)`