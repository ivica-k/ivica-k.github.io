+++
title = "5.1 Get all donors"
chapter = true
weight = 100
+++

# Reading donors

Our application works great so far when adding new donors and donation events. To make it usable for an organization, 
for the purposes of building a user interface, we need to display those donors. Let's build an API endpoint that will
display all donors from the DynamoDB table.

Looking at it from the architecture standpoint, this is what we want to achieve:

![](/images/donor_signup_db_arch_reading.png)

If we transformed it into steps, they would look like so:

1. A user sends an HTTP `GET` request
2. API Gateway receives it and invokes the Lambda function
3. Lambda function receives it and performs a query on the DynamoDB table
4. Lambda function returns a response containing query results to the user

#### API design

Going with the [nouns to represent resources and verbs to represent actions](https://restfulapi.net/resource-naming/)
mantra we can deduce that in this particular case we have a collection of resources, or "donors".

I'll go with **`/donors`**.

Adding another API endpoint is very easy; just append these couple of lines to `~/serverless_workshop/savealife/app.py`:

{{<highlight python>}}
@app.route("/donors", methods=["GET"])
def donors_get():
    db_response = get_app_db().donors_all()

    app.log.debug(f"DBResponse: {db_response}")

    return Response(
        body=asdict(db_response),
        status_code=200 if db_response.success else 400
    )
{{</highlight>}}

On the database front the things are almost as simple. The `donors_all()` function from `chalicelib/db_donor.py` may look
like this:

{{<highlight python "hl_lines=4-7">}}
    def donors_all(self):
        db_response = DBResponse(resource_id="", success=False, error_message="", return_value={})
        try:
            response = self._table.scan(
                FilterExpression=Key("PK").begins_with("DONOR"),
                ReturnConsumedCapacity="TOTAL",
            )

            donors = response.get("Items")

            self._logger.debug(f"Consumed {response.get('ConsumedCapacity').get('CapacityUnits')} capacity units")
            self._logger.debug(f"Fetched {len(donors)} donor(s)")

            if response.get("ResponseMetadata").get('HTTPStatusCode') == 200:
                db_response.success = True
                db_response.return_value = donors

            return db_response

        except Exception as exc:
            db_response.success = False
            db_response.error_message = str(exc)

            self._logger.exception(exc)

        finally:
            return db_response
{{</highlight>}}

Don't forget the import `from boto3.dynamodb.conditions import Key`

The highlighted part is the most interesting so let's go over it with a bit more detail. While inserting data with previous
functions we relied on the `put_item()` function to write data to DynamoDB. Reading data from DynamoDB can be achieved
by using:

- `Scan` operations
- `Query` operations

`Scan` operations are less efficient because they always scan the entire table or index. Optionally filters can be applied
to the data set but they are only applied after reading (scanning) all data first. In SQL terms you can think of them as:

![](/images/code_screenshots/70_100_1.png)

They can return up to 1MB of data and if more data is to be returned it is then paged.

`Query` operations are much more precise than `Scan` operations, and to be precise they require us to provide the
primary key value to query with. In SQL terms you can think of them as:

![](/images/code_screenshots/70_100_2.png)

Looking back to our code, we can see that it performs a `Scan` operation ($$$) with some server-side filtering; we
are asking the table to give us all the items whose primary key `PK` begins with `DONOR`. Let's see it in action:

```bash
http -b GET :8000/donors
```
will output something similar to:

![](/images/code_screenshots/70_100_3.png)

**Deploy at will** and verify that your function is returning data when running locally and in AWS.

`chalice deploy`

![](/images/code_screenshots/70_100_4.png)

Performing that `GET` request to list all donors is now working as it should, showing all donors
`http -b GET $(chalice url)/donors`

![](/images/code_screenshots/70_100_5.png)

Great success!