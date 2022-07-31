+++
title = "5.2 Get all donations"
chapter = true
weight = 200
+++

# Reading donations

Our application works great so far when adding new donors and donation events. To make it usable for an organization, 
for the purposes of building a user interface, we need to display donations. Let's build an API endpoint that will
display all donations from the DynamoDB table.

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

`http -b GET $(chalice url)/donations`

When you finish making code changes, and assuming you have at least one donation event created, invoking the API endpoint
should return results similar to these:

```bash
{
    "error_message": "",
    "resource_id": "",
    "return_value": [
        {
            "PK": "DONATION#7ca285890a7a",
            "address": "2nd street",
            "city": "Haarlem",
            "datetime": "2022-04-06T13:00:00"
        },
        {
            "PK": "DONATION#7d54f0be39c7",
            "address": "Main street",
            "city": "Haarlem",
            "datetime": "2022-04-06T12:00:00"
        }
    ],
    "success": true
}
```

Another great success!

***

{{%expand "Solution" %}}
Add this to `app.py`

```python
@app.route("/donations", methods=["GET"])
def donations_get():
    db_response = get_app_db().donations_all()

    app.log.debug(f"DBResponse: {db_response}")

    return Response(
        body=asdict(db_response),
        status_code=200 if db_response.success else 400
    )
```

and this to `chalicelib/db_donation.py`

{{<highlight python "hl_lines=4 9-35">}}
from uuid import uuid4
from . import DBResponse

from boto3.dynamodb.conditions import Key


class DonationMixin:

    def donations_all(self):
        db_response = DBResponse(resource_id="", success=False, error_message="", return_value={})
        try:
            response = self._table.scan(
                FilterExpression=Key("PK").begins_with("DONATION"),
                ReturnConsumedCapacity="TOTAL",
            )

            donations = response.get("Items")

            self._logger.debug(f"Consumed {response.get('ConsumedCapacity').get('CapacityUnits')} capacity units")
            self._logger.debug(f"Fetched {len(donations)} donation(s)")

            if response.get("ResponseMetadata").get('HTTPStatusCode') == 200:
                db_response.success = True
                db_response.return_value = donations

            return db_response

        except Exception as exc:
            db_response.success = False
            db_response.error_message = str(exc)

            self._logger.exception(exc)

        finally:
            return db_response
{{</highlight>}}

{{% /expand%}}