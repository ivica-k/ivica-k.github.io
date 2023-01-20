+++
title = "4.2 DB response"
chapter = true
weight = 200
+++

# DB response

Our database layer functions did a couple of things right but what is missing is a structured return statement, almost
like a contract. When the function executes successfully the API layer and our end-user received the value `True`. Not
the most user-friendly response for sure.

## Improving DB layer returns

A good place to start is by defining how the response will look like. It needs to convey several messages back to the
API layer/user:

- general success/failure when performing actions
- an error message if an error happens
- a `resource_id`, if there was a write to the database (like a donor sign-up)
- a data structure to contain items read from a database (a list of donors for example)

```json
{
  "resource_id": "",
  "error_message": "",
  "success": true,
  "return_value": {}
}
```

I've been a fan of Python `dataclasses` since they were introduced back in Python 3.6 and the above-mentioned data structure
is perfect for a dataclass.

#### Code changes

The `DBResponse` dataclass will be shared between all database mixins so a good place to create it is in `chalicelib/__init__.py`

```python
from dataclasses import dataclass


@dataclass
class DBResponse:
    resource_id: str
    success: bool
    error_message: str
    return_value: dict
```

Using it in `chalicelib/db_donor.py` would go like this:

{{<highlight python "hl_lines=2 11 23-26 29-30 34-35">}}
from uuid import uuid4
from . import DBResponse


class DonorMixin:

    def donor_signup(self, donor_dict):
        uid = str(uuid4()).split("-")[-1]
        db_response = DBResponse(resource_id="", success=False, error_message="", return_value={})

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
            self._logger.debug(f"Inserted donor '{donor_dict.get('email')}' into DynamoDB table '{self._table}'")

            db_response.success = True
            db_response.resource_id = uid

            return db_response

        except Exception as exc:
            db_response.success = False
            db_response.error_message = str(exc)

            self._logger.exception(exc)

        finally:
            return db_response
{{</highlight>}}

## DIY

Please proceed to implement the `DBResponse` dataclass for the `DonationMixin`.

***

{{%expand "Solution" %}}
Add this to `chalicelib/db_donation.py`

{{<highlight python>}}
from uuid import uuid4
from . import DBResponse


class DonationMixin:
    def donation_create(self, donation_dict):
        uid = str(uuid4()).split("-")[-1]
        db_response = DBResponse(resource_id="", success=False, error_message="", return_value={})

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

            db_response.success = True
            db_response.resource_id = uid

            return db_response

        except Exception as exc:
            db_response.success = False
            db_response.error_message = str(exc)

            self._logger.exception(exc)

        finally:
            return db_response

{{</highlight>}}

{{% /expand%}}