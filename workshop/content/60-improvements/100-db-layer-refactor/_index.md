+++
title = "4.1 DB layer refactoring"
chapter = true
weight = 100
+++

# DB layer refactoring

Our DB layer is currently the biggest piece of code judging by the number of lines, without any tendency to stop growing.
Let's refactor it by splitting it up before it gets too big.

## Splitting by responsibility

There are obviously two (for now) types of items we're working with; a "donor" and a "donation". Although we decided
that we want to keep them together in a single database that does not mean that the code that manages them needs to be
in a single file.

#### Donor mixin

Mixins in Python are a great way to achieve multiple inheritance, split code logically and be one step closer to stop
repeating too much of your code.

Create a new file for our `DonorMixin`: `chalicelib/db_donor.py`. Move the donor related code from `db.py` to the newly
created file.

{{<highlight python>}}
from uuid import uuid4


class DonorMixin:
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

{{</highlight>}}

#### Donation

We can repeat pretty much everything we did to the donor related code from `chalicelib/db.py`. Create `chalicelib/db_donation.py`
and move all relevant code pieces to it.

{{<highlight python>}}
from uuid import uuid4


class DonationMixin:
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

{{</highlight>}}

### Bring it all together

Our `chalicelib/db.py` should now be much smaller and easier to work with. This is what it looks like after refactoring:

{{<highlight python "hl_lines=13 14 37">}}
import logging
from os import getenv

import boto3

try:
    from dotenv import load_dotenv

    load_dotenv()
except ImportError:
    pass

from .db_donation import DonationMixin
from .db_donor import DonorMixin


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


class SavealifeDB(DonorMixin, DonationMixin):
    def __init__(self, table, logger):
        self._table = table
        self._logger = logger

{{</highlight>}}

Highlighted lines are responsible for importing and using the mixins we created just a few minutes ago.