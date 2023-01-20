+++
title = "2.4 Persist the donor to DynamoDB"
chapter = true
weight = 400
+++

# Persist the donor to DynamoDB

On the [donor sign-up](../30-donor-signup/300-donor-signup.html) page we saw how to build and deploy a simple 
AWS Lambda function using Chalice that parses the JSON payload and returns it to the caller. On this page we are going 
to take that JSON payload and save it to a database.

Looking at it from the architecture standpoint, this is what we want to achieve:

![](/images/donor_signup_db_arch.png)

If we transformed it into steps, they would look like so:

1. A user sends an HTTP `POST` request with a JSON payload
2. API Gateway receives it and invokes the Lambda function
3. Lambda function receives the JSON payload in the `event` argument and saves it to a database (with or without changes)

## What we have so far?

If you have been following along, your `~/serverless_workshop/savealife/app.py` should have the following contents:

```python
import logging

from os import getenv
from chalice import Chalice

try:
    from dotenv import load_dotenv

    load_dotenv()
except ImportError:
    pass

first_name = getenv("WORKSHOP_NAME", "ivica")  # replace with your own name of course

app = Chalice(app_name=f"{first_name}-savealife")
app.log.setLevel(logging.DEBUG)

@app.route("/donor/signup", methods=["POST"])
def donor_signup():
    body = app.current_request.json_body

    app.log.debug(f"Received JSON payload: {body}")
    app.log.info("This is a INFO level message")

    return body
```

## Persisting the JSON payload to a DynamoDB table

DynamoDB provides an HTTP API to perform actions - there are no persistent TCP connections, pools or anything similar.
Want to save an item? Sure, there's a [PutItem API request](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_PutItem.html)
that you can make. We of course will not be simple caveman making raw API requests. We will be using the very powerful 
AWS SDK for Python called [Boto](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html).

Your `requirements.txt` file should have `boto3` in it.
```bash{linenos=false}
cat requirements.txt 
```
will result in something similar to:

```bash{linenos=false}
boto3==1.24.75
botocore>=1.25.2
```

Fun fact: Do you have an idea why is it called "boto"? Click [here](https://en.wikipedia.org/wiki/Amazon_river_dolphin) to find out.

***

#### Chalice multifile support

Before we proceed any further, a little digression. Chalice as a framework allows us to ship 
[arbitrary assets](https://aws.github.io/chalice/topics/multifile.html) (JSON files, additional Python files etc.) with 
our Lambda function. Whatever files are placed in the `chalicelib` directory will be recursively included in the deployment.

The `chalicelib/` folder is to be located on the same level as your `app.py`:

```bash{linenos=false}
.
├── app.py
├── chalicelib
│   └── __init__.py
└── requirements.txt
```

We can leverage this functionality to split up our application into multiple files as it grows. It is the perfect place
to put our DynamoDB related code into.

***

#### DynamoDB code

Add a file called `db.py` to `~/serverless_workshop/savealife/chalicelib/` with the following contents:

```python
import logging
from os import getenv

import boto3

try:
    from dotenv import load_dotenv

    load_dotenv()
except ImportError:
    pass

ENV = getenv("ENV", "dev")
first_name = getenv("WORKSHOP_NAME", "ivica")  # replace with your own name of course

logger = logging.getLogger(f"{first_name}-savealife")

_DB = None
TABLE_NAME = getenv("TABLE_NAME")


def get_app_db():
    global _DB

    if _DB is None:
        _DB = SavealifeDB(
            table=boto3.resource("dynamodb").Table(TABLE_NAME), logger=logger
        )

    return _DB


class SavealifeDB:
    def __init__(self, table, logger):
        self._table = table
        self._logger = logger

    def donor_signup(self, donor_dict):
        try:
            self._table.put_item(
                Item={
                    "first_name": donor_dict.get("first_name"),
                    "city": donor_dict.get("city"),
                    "type": donor_dict.get("type"),
                    "email": donor_dict.get("email"),
                }
            )
            self._logger.debug(
                f"Inserted donor '{donor_dict.get('email')}' into DynamoDB table '{self._table}'"
            )

            return True

        except Exception as exc:
            self._logger.exception(exc)

```

Let's look at the `SavealifeDB` class. The `donor_signup` method is what interacts with the DynamoDB HTTP API. 
Our `table` resource has a `put_item()` function which we use to... yeah, put an item into the table. The item itself 
is a just a dictionary.

#### App code changes

Our application code from `app.py` will be a bit bulkier since we added logging on the [logging page](../30-donor-signup/600-logging.html).
It looks like this:

{{<highlight python "hl_lines=5 26">}}
import logging

from os import getenv
from chalice import Chalice
from chalicelib.db import get_app_db

try:
    from dotenv import load_dotenv

    load_dotenv()
except ImportError:
    pass

first_name = getenv("WORKSHOP_NAME", "ivica")  # replace with your own name of course

app = Chalice(app_name=f"{first_name}-savealife")
app.log.setLevel(logging.DEBUG)


@app.route("/donor/signup", methods=["POST"])
def donor_signup():
    body = app.current_request.json_body

    app.log.debug(f"Received JSON payload: {body}")

    return get_app_db().donor_signup(body)

{{</highlight>}}

With the first highlighted line we import the `get_app_db` function that is the interface for interacting with DynamoDB. Changing the `return` statement _will_ break our tests but that's all right.

Invoke the local endpoint and you will get a result similar to this:

```bash{linenos=false}
{
    "Code": "InternalServerError",
    "Message": "An internal server error occurred."
}
```

We can also see that the `chalice local` shows a stacktrace because an error happened

```bash
ivica-savealife - ERROR - Caught exception for path /donor/signup
Traceback (most recent call last):
  File "pythonlib/python3.10/site-packages/chalice/app.py", line 1752, in _get_view_function_response
    response = view_function(**function_args)
  ... SNIP ...
  File "pythonlib/python3.10/site-packages/boto3/resources/base.py", line 125, in __init__
    raise ValueError(f'Required parameter {identifier} not set')
ValueError: Required parameter name not set
```

Formatting the message in a different way, `ValueError: Required parameter 'name' not set` for example,
would make it much more clear: a required parameter `name` is not set. In our case, name of the DynamoDB table is
not set.