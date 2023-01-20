+++
title = "3.3 Coding single table design"
chapter = true
weight = 300
+++

Some contents of this page was copied or inspired by [Alex DeBries article](https://www.alexdebrie.com/posts/dynamodb-single-table/)
on single table design. Alex is the authority on DynamoDB and his [DynamoDB book](https://www.dynamodbbook.com/) is 
a must-read for anyone designing data models on DynamoDB.

# Coding single table design

On the previous page we discussed how to approach storing different types of data with our application, and the answer
to that question is: by using a DynamoDB table with a single table design approach.

![](/images/db_table_2.png)

This page will deal with code changes needed for this design to work.

## Table changes

It is not possible to change DynamoDB table keys after the table was created. Since we're still in the development phase
of our application it is OK to simply re-create the table.

To delete the existing table please execute:

```bash
aws dynamodb delete-table --table-name $WORKSHOP_NAME-savealife-$ENV --profile workshop
```

Now re-create your table with:

```bash
aws dynamodb create-table --table-name $WORKSHOP_NAME-savealife-$ENV \
  --profile workshop \
  --attribute-definitions AttributeName=PK,AttributeType=S \
  --key-schema AttributeName=PK,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

The end result will be the same.

## DB layer changes

Let's talk about the DB layer changes in `chalicelib/db.py`. What changed with our table and how should our DB layer
reflect those changes?

A "donor" item looked like this, with `first_name` being the primary key:

![](/images/code_screenshots/50_300_1.png)

Now the primary key will be the `PK` field and everything else, the whole JSON payload, will become attributes.

You may use the following code sample to implement the required changes:

{{<highlight python "hl_lines=3 40 48 61 68">}}
import logging
from os import getenv
from uuid import uuid4

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
        uid = str(uuid4()).split("-")[-1]
        try:
            self._table.put_item(
                Item={
                    "first_name": donor_dict.get("first_name"),
                    "city": donor_dict.get("city"),
                    "type": donor_dict.get("type"),
                    "email": donor_dict.get("email"),
                    "PK": f"DONOR#{uid}",
                }
            )
            self._logger.debug(
                f"Inserted donor '{donor_dict.get('email')}' into DynamoDB table '{self._table}'"
            )

            return True

        except Exception as exc:
            self._logger.exception(exc)

    def donation_create(self, donation_dict):
        uid = str(uuid4()).split("-")[-1]
        try:
            self._table.put_item(
                Item={
                    "city": donation_dict.get("city"),
                    "datetime": donation_dict.get("datetime"),
                    "address": donation_dict.get("address"),
                    "PK": f"DONATION#{uid}",
                }
            )
            self._logger.debug(
                f"Inserted donation '{donation_dict.get('city')}, {donation_dict.get('address')}' "
                f"into DynamoDB table '{self._table}'"
            )

            return True

        except Exception as exc:
            self._logger.exception(exc)

            return False

{{</highlight>}}

Highlighted lines are the needed changes, and we only need a few. Each "donor" and each "donation" must be unique, so
we reach out to the trusty `uuid4()` for that. You have also probably noticed that both the "donor" and the "donation" item 
have the `PK` field added to them with a unique string.

How unique are `uuid4` strings? We would need to generate 1 billion UUIDs
per second for 85 years to have a 50% chance of generating two identical UUIDs. Unique enough for me. [Source](https://en.wikipedia.org/wiki/Universally_unique_identifier#Collisions)

## Deploy and verify it works

We went through a lot of text and a lot of code. Now's the time to see the fruits of our labor: `chalice deploy` time.

```bash
# add a donor
http -b POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com" type="A+" city="Amsterdam"

# add a donation
http -b POST $(chalice url)/donation/create city=Haarlem address="Main street" datetime="2022-04-06T12:00:00"
```

Table explorer in the DynamoDB UI shows that both items were inserted into the single table we have.

![](/images/single_table_donor_donation.png)

***

As we saw on the [DynamoDB 101](../40-persisting-data/200-dynamodb-101.html) page, Alex DeBrie's resources are a gold mine. 
Watch [part 1](https://www.youtube.com/watch?v=fiP2e-g-r4g) and [part 2](https://www.youtube.com/watch?v=0uLF1tjI_BI) 
of his "Data modeling with DynamoDB presentation" for more information on single table design, access patterns and more.