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
```bash
TypeError: Object of type DBResponse is not JSON serializable
```

#### In action locally

Notice the `HTTP/1.1 200 OK` signifing that the `status_code` is 200.
```bash
http POST :8000/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"
HTTP/1.1 200 OK
Content-Length: 82
Content-Type: application/json
Date: Sat, 08 Oct 2022 15:01:09 GMT
Server: BaseHTTP/0.6 Python/3.10.7

{
    "error_message": "",
    "resource_id": "e6dd12d96578",
    "return_value": {},
    "success": true
}
```

Raising an exception with `raise Exception("This is a simulated exception")` results in:

```bash
http POST :8000/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"
HTTP/1.1 400 Bad Request
Content-Length: 112
Content-Type: application/json
Date: Sat, 08 Oct 2022 15:03:57 GMT
Server: BaseHTTP/0.6 Python/3.10.7

{
    "error_message": "This is a simulated exception",
    "resource_id": "43d3ac7db857",
    "return_value": {},
    "success": false
}
```

#### In action on AWS

`chalice deploy` and let's see what happens when we invoke the `/donor/signup` endpoint:

```bash
http POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 82
Content-Type: application/json
Date: Sat, 08 Oct 2022 15:27:52 GMT
Via: 1.1 4445c4223f8c2460ef5d29a08d1cc6ac.cloudfront.net (CloudFront)
X-Amz-Cf-Id: m5BQPAIkT-q0So0xw8RTikOw0gKaRt7etH9rAeuIXe3huOILE3DXUw==
X-Amz-Cf-Pop: AMS54-C1
X-Amzn-Trace-Id: Root=1-634196f6-0813e73e7bf13966216b7bdf;Sampled=0
X-Cache: Miss from cloudfront
x-amz-apigw-id: ZsSGeHgOliAFepw=
x-amzn-RequestId: 4d934415-495a-4e90-9ac2-6dd314d708b4

{
    "error_message": "",
    "resource_id": "7838aa6dbda1",
    "return_value": {},
    "success": true
}
```

And to simulate an error I have deleted the DynamoDB table that is used by the application.

```bash
http POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"
HTTP/1.1 400 Bad Request
Connection: keep-alive
Content-Length: 181
Content-Type: application/json
Date: Sat, 08 Oct 2022 15:28:57 GMT
Via: 1.1 c149c6b8a4d6f497cac6f2d9e9e6be40.cloudfront.net (CloudFront)
X-Amz-Cf-Id: RZjn71PQLMWsgoNAgf1MxBKSN2XilwC17JXOEkcINlEnoPkIzmepBA==
X-Amz-Cf-Pop: AMS54-C1
X-Amzn-Trace-Id: Root=1-63419739-6dcc700a51e58f5807eed30f;Sampled=0
X-Cache: Error from cloudfront
x-amz-apigw-id: ZsSQ8E3tFiAFm2A=
x-amzn-RequestId: 623cd6b7-74a7-4c03-b73a-f684cf1eb5e0

{
    "error_message": "An error occurred (ResourceNotFoundException) when calling the PutItem operation: Requested resource not found",
    "resource_id": "",
    "return_value": {},
    "success": false
}
```