---
title: "6.9 Get all donors from a city - for real"
chapter: true
weight: 90
---

# Get all donors from a city for real

Our goal when we started this chapter was to fetch all donors for a specific city. Secondary indexes allow us to perform
the needed query and the DynamoDB streams are "telling" us when to perform that query.

This page will go through the necessary code changes to get all donors from a city when a new donation is created.

#### Query the index

The `Query` operation we should be executing looks like this and goes into `db_donor.py`:

{{< highlight python "hl_lines=5 6" >}}
    def donors_by_city(self, city):
        db_response = DBResponse(resource_id="", success=False, error_message="", return_value={})
        try:
            response = self._table.query(
                IndexName="SK-PK-index",
                KeyConditionExpression="SK = :SK AND begins_with(PK, :PK)",
                ExpressionAttributeValues={
                    ":SK": f"CITY#{city}",
                    ":PK": "DONOR#",
                },
                ReturnConsumedCapacity="TOTAL",
            )

            self._logger.debug(f"Consumed {response.get('ConsumedCapacity').get('CapacityUnits')} capacity units")
            self._logger.debug(f"Retrieved all donors from city '{city}'")

            if response.get("ResponseMetadata").get('HTTPStatusCode') == 200:
                db_response.success = True
                db_response.return_value = response.get("Items")

            return db_response

        except Exception as exc:
            db_response.success = False
            db_response.error_message = str(exc)

            self._logger.exception(exc)

        finally:
            return db_response
{{</highlight>}}

No surprises there other than the highlighted lines, which show that we are indeed querying the index named `SK-PK-index` 
and how the `begins_with()` can be used to narrow down the result set.

API changes will for now only log the results:
{{< highlight python "hl_lines=6-9" >}}
@app.on_dynamodb_record(stream_arn=stream_arn)
def handle_stream(event):
    for record in event:
        stream_data = record.new_image

        city_name = stream_data.get("city").get("S")
        db_response = get_app_db().donors_by_city(city_name)

        app.log.debug(db_response)
{{</highlight>}}

After deploying these changes and adding a new donation - you guessed it - a permissions error:
![](/images/code_screenshots/80_90_1.png)

Any ideas on how we can solve this one?

{{%expand "Solution" %}}
Add the following statement to `policy-handle-stream.json`

```json
{
  "Action": [
    "dynamodb:Query"
  ],
  "Resource": [
    "arn:aws:dynamodb:eu-central-1:932785857088:table/YOUR_NAME_HERE-savealife-dev/index/*"
  ],
  "Effect": "Allow"
}
```

Don't forget to deploy :)
{{% /expand%}}

After fixing the permissions and deploying, add a new donation event in Haarlem. If everything worked well we should be 
getting back two donors that live in Haarlem.

```bash
http -b POST $(chalice url)/donation/create city=Haarlem address="Other street" datetime="2022-04-06T12:00:00"
```

and the result:

![](/images/code_screenshots/80_90_2.png)

or with better formatting:

![](/images/code_screenshots/80_90_3.png)

Getting email addresses from this result set and sending them en email will be our next step. To infinity, and beyond!