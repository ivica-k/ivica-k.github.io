+++
title = "5.3 Get a single donor"
chapter = true
weight = 300
+++

# Reading a single donor

Our application works great so far when adding new donors and donation events. To make it usable for an organization, 
for the purposes of building a user interface, we need to display details about a single donor.
Let's build an API endpoint that will display details about a single donor from the DynamoDB table.

Looking at it from the architecture standpoint, this is what we want to achieve:

![](/images/donor_signup_db_arch_reading.png)

If we transformed it into steps, they would look like so:

1. A user sends an HTTP `GET` request
2. API Gateway receives it and invokes the Lambda function
3. Lambda function receives it and performs a query on the DynamoDB table
4. Lambda function returns a response containing query results to the user

#### Forethought

Let's think a bit on how to build this one before we dive into API design. We would like to get a single donor item
from the table. If you remember from our discussion on reading vs. scanning data, we need a `Query` operation here - 
they are precise and require the exact value of the primary key (`PK` in our case).

On one of the [previous pages](../70-reading-data/100-donors.html) we saw that the primary key for a donor is for example 
`DONOR#68252cff869a`. That's our target then: we need a way to somehow perform a query that may look like this in the
SQL world:

![](/images/code_screenshots/70_300_1.png)

#### API design

There are multiple ways of creating a `GET` endpoint that receives a dynamic value with Chalice. One of them is using
query-strings; we've all seen URLs that look like this: `https://myweather.com/api?city=Florence`. `city=Florence` is
the query string.

If we applied this to our use case we could have an endpoint looking like `/donor?id=1234`.

Another way of creating such an endpoint is to append the dynamic value to the endpoint itself: `/donor/1234`. My personal
preference is to go with the second approach, where the dynamic values is a part of the URL.

**`/donor/1234`** it is then.

#### API layer changes

```python
@app.route("/donor/{donor_id}", methods=["GET"])
def donor_by_id(donor_id):
    db_response = get_app_db().donor_by_id(donor_id)

    app.log.debug(f"DBResponse: {db_response}")

    return Response(
        body=asdict(db_response),
        status_code=200 if db_response.success else 400
    )
```

Code changes on the API layer are quite small. The only real difference is visible on the first line. We introduced
a dynamic value of `donor_id` that will be supplied by the client when they invoke the endpoint. Whatever that value is,
it will be passed to our function `donor_by_id(donor_id)`

#### DB layer changes

Most of the function body is quite similar to the two we added previously; however, there are changes in the way that the 
database action is performed:

{{< highlight python "hl_lines=4-10 17" >}}
    def donor_by_id(self, donor_id):
        db_response = DBResponse(resource_id="", success=False, error_message="", return_value={})
        try:
            response = self._table.query(
                KeyConditionExpression="PK = :PK",
                ExpressionAttributeValues={
                    ":PK": f"DONOR#{donor_id}"
                },
                ReturnConsumedCapacity="TOTAL",
            )

            self._logger.debug(f"Consumed {response.get('ConsumedCapacity').get('CapacityUnits')} capacity units")
            self._logger.debug(f"Retrieved donor with id {donor_id}")

            if response.get("ResponseMetadata").get('HTTPStatusCode') == 200:
                db_response.success = True
                db_response.return_value = response.get("Items")[0]

            return db_response

        except Exception as exc:
            db_response.success = False
            db_response.error_message = str(exc)

            self._logger.exception(exc)

        finally:
            return db_response
{{</highlight>}}

The first highlighted part shows a `Query` operation with two distinct parts:

```python
KeyConditionExpression="PK = :PK"
# and 
ExpressionAttributeValues={
    ":PK": f"DONOR#{donor_id}"
}
```

When read in plain English, `KeyConditionExpression="PK = :PK"` simply means "primary key `PK` equals the placeholder 
value of `:PK`". That placeholder value is then supplied by the `ExpressionAttributeValues` where the value for our `:PK`
placeholder is set to `DONOR#68252cff869a`. Basically this

![](/images/code_screenshots/70_300_1.png)

but in DynamoDB query language.

#### Together

Putting the two code additions together we end up with the following action working just fine:

`http -b GET :8000/donor/68252cff869a`

![](/images/code_screenshots/70_300_2.png)