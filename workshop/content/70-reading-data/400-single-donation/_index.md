+++
title = "5.4 Get a single donation"
chapter = true
weight = 400
+++

# Reading a single donation

Our application works great so far when adding new donors and donation events. To make it usable for an organization, 
for the purposes of building a user interface, we need to display donation details. Let's build an API endpoint that will
display donation details from the DynamoDB table.

Looking at it from the architecture standpoint, this is what we want to achieve:

![](/images/donor_signup_db_arch_reading.png)

If we transformed it into steps, they would look like so:

1. A user sends an HTTP `GET` request
2. API Gateway receives it and invokes the Lambda function
3. Lambda function receives it and performs a query on the DynamoDB table
4. Lambda function returns a response containing query results to the user

## Your time to shine again!

Code changes you should make are very similar to the ones we made on the previous page. A bit of copy-pasting
never hurt anyone, right?

## End results

`http -b GET :8000/donation/xxxxxxxx`

When you finish making code changes, and assuming you have at least one donation event created, invoking the API endpoint
should return results similar to these:

![](/images/code_screenshots/70_400_1.png)

Another great success!

***

{{%expand "Solution" %}}
Add this to `app.py`

```python
@app.route("/donation/{donation_id}", methods=["GET"])
def donation_by_id(donation_id):
    db_response = get_app_db().donation_by_id(donation_id)

    app.log.debug(f"DBResponse: {db_response}")

    return Response(
        body=asdict(db_response),
        status_code=200 if db_response.success else 400
    )
```

and this to `chalicelib/db_donation.py`

{{<highlight python "hl_lines=9-36">}}
from uuid import uuid4
from . import DBResponse

from boto3.dynamodb.conditions import Key


class DonationMixin:

    def donation_by_id(self, donation_id):
        db_response = DBResponse(resource_id="", success=False, error_message="", return_value={})
        try:
            response = self._table.query(
                KeyConditionExpression="PK = :PK",
                ExpressionAttributeValues={
                    ":PK": f"DONATION#{donation_id}"
                },
                ReturnConsumedCapacity="TOTAL",
            )

            self._logger.debug(f"Consumed {response.get('ConsumedCapacity').get('CapacityUnits')} capacity units")
            self._logger.debug(f"Retrieved donation with id {donation_id}")

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

{{% /expand%}}