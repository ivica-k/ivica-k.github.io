+++
title = "4.3 API response"
chapter = true
weight = 300
+++

# API response

Returning a `dataclass` from our DB layer to the API layer establishes a contract between these two pieces of code. Any
time they want to communicate, the contract needs to be respected. 

{{%expand "APIs are forever" %}}
![](/images/api_forever.png)
{{% /expand%}}

But the same type of contract should exist between the API layer and its users. That way the users know what to expect
as a response to their requests. We could go ahead and implement the [response data mapping reference](https://docs.aws.amazon.com/apigateway/latest/developerguide/request-response-data-mappings.html)
but as always, simple is better than complex.

Our API response does not need to add anything to the `DBResponse` class since it contains everything that the end-user
(browser) will need. Everything except status code. 

Our API must 
[handle errors gracefully and return standard error codes.](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/#h-handle-errors-gracefully-and-return-standard-error-codes)

{{<highlight python "hl_lines=4 6 27 31-34 43 47-50">}}
import logging

from os import getenv
from chalice import Chalice, Response
from chalicelib.db import get_app_db
from dataclasses import asdict

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

    db_response = get_app_db().donor_signup(body)

    app.log.debug(f"DBResponse: {db_response}")

    return Response(
        body=asdict(db_response),
        status_code=200 if db_response.success else 400
    )


@app.route("/donation/create", methods=["POST"])
def donation_create():
    body = app.current_request.json_body

    app.log.debug(f"Received JSON payload: {body}")

    db_response = get_app_db().donation_create(body)

    app.log.debug(f"DBResponse: {db_response}")

    return Response(
        body=asdict(db_response),
        status_code=200 if db_response.success else 400
    )

{{</highlight>}}

Shown above is an example of using the Chalice `Response` class to return our custom response. Since we agreed that the
`DBResponse` data structure contains all the needed data, the only variable in our case is the `status_code` field
that is highlighted. We rely on the `DBResponse.success` attribute to tell us whether the database operation was 
successful and our `status_code` reflects that.

Python dataclasses are not JSON-serializable so if you forgot to wrap it into a `asdict()` function, a `TypeError` will
be raised:
![](/images/code_screenshots/60_300_1.png)

#### In action locally

Notice the `HTTP/1.1 200 OK` signifing that the `status_code` is 200.
```bash
http POST :8000/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"
```

resulting in:
![](/images/code_screenshots/60_300_2.png)

Raising an exception with `raise Exception("This is a simulated exception")`:

```bash
http POST :8000/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"
```

results in:
![](/images/code_screenshots/60_300_3.png)

#### In action on AWS

`chalice deploy` and let's see what happens when we invoke the `/donor/signup` endpoint:

```bash
http POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"
```

![](/images/code_screenshots/60_300_4.png)

And to simulate an error I have deleted the DynamoDB table that is used by the application.

```bash
http POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"
```

results in:
![](/images/code_screenshots/60_300_5.png)