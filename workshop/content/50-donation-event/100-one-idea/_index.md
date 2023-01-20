+++
title = "3.1 One idea..."
chapter = true
weight = 100
+++

# One idea...

One idea we could try is to implement the "Create a donation event" feature in the same way as our donor sign-up feature.

Architecturally they would look the same:

![](/images/donor_signup_db_arch.png)

If we transformed it into steps, it would also be:

1. A user sends an HTTP `POST` request with a JSON payload
2. API Gateway receives it and invokes the Lambda function
3. Lambda function receives the JSON payload in the `event` argument

#### API design and Lambda function

I'll go with **`/donation/create`**.

Our `~/serverless_workshop/savealife/app.py` file can be extended to include the highlighted lines:

{{<highlight python "hl_lines=29-39">}}
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


@app.route("/donation/create", methods=["POST"])
def donation_create():
    body = app.current_request.json_body

    app.log.debug(f"Received JSON payload: {body}")

    return get_app_db().donation_create(body)
{{</highlight>}}

When it comes to the JSON payload itself I would propose we keep things simple again. What potential donors
should know about a blood donation are 4 pieces of information: in which _city_ is it being held, on _which date_ 
and at _what time_, which _blood types_ are needed and what is the _location_.
So, something like this:

```json
{
  "city": "Haarlem",
  "datetime": "2022-6-4 13:00:00",
  "blood_types": "A-",
  "address": "Main Street"
}
```

#### Persisting to DynamoDB

Very similar to how we copy-pasted existing code for the Lambda function, our blood donation persistence layer 
can be copy-pasted from the `donor_signup` function. Add the highlighted lines to 
`~/serverless_workshop/savealife/chalicelib/db.py`

{{<highlight python "hl_lines=6-23">}}
class SavealifeDB:
    def __init__(self, table, logger):
        # ... SNIP ...


    def donation_create(self, donation_dict):
        try:
            self._table.put_item(
                Item={
                    "city": donation_dict.get("city"),
                    "datetime": donation_dict.get("datetime"),
                    "address": donation_dict.get("address")
                }
            )
            self._logger.debug(
                f"Inserted donation '{donation_dict.get('city')}, {donation_dict.get('address')}' "
                f"into DynamoDB table '{self._table}'"
            )

            return True

        except Exception as exc:
            self._logger.exception(exc)
{{</highlight>}}

##### Moment of truth

Run the local server `chalice local --autoreload` and hit the endpoint we just created. Does it work? What happened?

#### Another error ...

The endpoint, from the API perspective, works just fine but nothing will be saved to the DynamoDB table. That is because
of how our DynamoDB table was created:

{{<highlight python>}}
client.create_table(
    TableName=table_name,
    AttributeDefinitions=[{"AttributeName": "first_name","AttributeType": "S"}],
    KeySchema=[{"AttributeName": "first_name", "KeyType": "HASH"}]
    ... SNIP ...
)
{{</highlight>}}

Having logging configured help us to clearly see the issue; `first_name` is our primary key and the expected attribute.

```bash
http -b POST :8000/donation/create city=Haarlem address="Main street"
```
will show something along the lines of:

```bash{linenos=false}
# in another terminal
chalice local --autoreload
Serving on http://127.0.0.1:8000
ivica-savealife - DEBUG - Received JSON payload: {'city': 'Haarlem', 'address': 'Main street'}
ivica-savealife - ERROR - An error occurred (ValidationException) when calling the PutItem operation: One or more parameter values were invalid: Missing the key first_name in the item
```

Back to the drawing board...

![](/images/drawing_board.webp)