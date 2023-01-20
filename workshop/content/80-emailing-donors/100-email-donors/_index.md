---
title: "6.10 Email donors"
chapter: true
weight: 100
---

# Email donors

We covered so much in this chapter and the last step is to build the emailing functionality which will notify donors 
from a specific city whenever there's a new blood donation event planned in their city.

As described in the chapter title, we will be using AWS Simple Email Service to send out emails because it is easy to use
from `boto3` and offers 62.000 emails for free per month.

The solution roughly looks like this, and we are now at the middle step, which will batch up to 50 email addresses
(a limitation of AWS SES) and send a message to them.

![](/images/email_donors_streams.png)

#### SES configuration

Even though using SES is relatively simple with `boto3`, the service itself comes with certain limitations when you first
start using it. AWS calls this the "sandbox mode".

From the docs:

> In a sandbox environment, you can use all of the features offered by Amazon SES; however, certain sending limits 
> and restrictions apply. When you’re ready to move out of the sandbox, submit a request for production access

In practice this means that:
- we can send up to 200 emails in a 24 hour period
- we can send up to 1 email per second
- we can only use a "Verified identity" as the `FROM` email address

All of these limitations are good enough for the purposes of this workshop. I have verified an identity in advance and 
it will be provided to you.

#### Batching up to 50 emails

I was never a great developer so [StackOverflow](https://stackoverflow.com/a/312464) helped me on this one. 
Create the `chalicelib/utils.py` file and add in it

```python
def chunk_list(lst, chunk_size):
    for i in range(0, len(lst), chunk_size):
        yield lst[i:i + chunk_size]
```

This will allow us to split a list `lst` with many elements into even chunks of `chunk_size`. This is how it works:

![](/images/code_screenshots/80_100_1.png)

We can also write a test for this function. Your time to shine again.

{{%expand "Solution" %}}
Create a file `tests/test_utils.py` and place the following code in it:

```python
from chalicelib.utils import chunk_list


def test_chunk_list():
    expected_value = [[1, 2], [3, 4], [5]]
    return_value = list(chunk_list([1, 2, 3, 4, 5], 2))

    assert expected_value == return_value
```
{{% /expand%}}

Run `pytest` to make sure it is included and passing:

![](/images/code_screenshots/80_100_2.png)

And finally, batching those email addresses can be done in this way:

{{<highlight python "hl_lines=9 11">}}
from chalicelib.utils import chunk_list
# ... SNIP ...

@app.on_dynamodb_record(stream_arn=stream_arn)
def handle_stream(event):
    # ... SNIP ...
    city_name = stream_data.get("city").get("S")
    db_response = get_app_db().donors_by_city(city_name)
    all_emails = [elem.get("email") for elem in db_response.return_value]
    
    batched_emails = list(chunk_list(all_emails, 50))
    
    app.log.debug(f"Gathered '{len(all_emails)}' from donors")
    
    for email_batch in batched_emails:
        # send email
{{</highlight>}}

#### Emailing donors using SES

Boto3 documentation for [sending an email with SES](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ses.html#SES.Client.send_email)
is clear and explicit (it even lists certain limitations of AWS SES sandbox mode).

It shows us how to use the SES client to send emails and that is exactly what we will do. Below are the necessary code 
changes:

{{<highlight python "hl_lines=1 3 8 10">}}
import boto3
# ... SNIP ...
ses_client = boto3.client("ses")

@app.on_dynamodb_record(stream_arn=stream_arn)
def handle_stream(event):
    # ... SNIP ...
    batched_emails = list(chunk_list(all_emails, 50))
    
    app.log.debug(f"Gathered '{len(all_emails)}' from donors")
    
    for email_batch in batched_emails:
        ses_client.send_email(
            Source="TO_BE_PROVIDED",
            Destination={
                "ToAddresses": email_batch
            },
            Message={
                "Subject": {
                    "Data": f"New blood donation event in {city_name}",
                    "Charset": "UTF-8",
                },
                "Body": {
                    "Html": {
                        "Data": f"<p>Dear donor,</p><p>There is a new blood donation event happening in {city_name}.</p><p>Come join us!</p>",
                        "Charset": "UTF-8",
                    }
                }
            }
        )
{{</highlight>}}

##### Permissions, permissions

As always, we have to allow our function to send emails. This is done by appending the following statement to the
`policy-handle-stream.json` file:

```json
{
  "Action": [
    "ses:SendEmail"
  ],
  "Resource": [
      "arn:aws:ses:eu-central-1:932785857088:identity/TO_BE_PROVIDED"
  ],
  "Effect": "Allow"
}
```

Deploy everything and add a new donation in Haarlem

```bash
chalice deploy
# ... SNIP ...
http -b POST $(chalice url)/donation/create city=Haarlem address="Other street" datetime="2022-04-06T12:00:00"
```

### Moment of truth

![](/images/email_received.png)

If you made it this far, congratulations :thumbsup: It hasn't been easy for sure.
