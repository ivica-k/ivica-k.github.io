---
title: "6.5 Reading stream data"
chapter: true
weight: 50
---

# Reading stream data

Our application needs to be configured to respond to events created by the DynamoDB stream. Chalice supports that with
the `@app.on_dynamodb_record` decorator. The decorator can be used in this way:

```python
@app.on_dynamodb_record(stream_arn="STREAM_ARN_HERE")
```

The stream ARN must be provided because it contains some automatically generated values that Chalice can not guess or
compute. That is why we added the `STREAM_ARN` to `.env` for local development, and on the previous page we configured
environment variables (including `STREAM_ARN`) per function in the `.chalice/config.json` file.

We need to make a code change in the `app.py` to read the variable:

{{<highlight python "hl_lines=2">}}
first_name = getenv("WORKSHOP_NAME", "ivica")  # replace with your own name of course
stream_arn = getenv("STREAM_ARN")

app = Chalice(app_name=f"{first_name}-savealife")
app.log.setLevel(logging.DEBUG)
{{</highlight>}}

#### Stream event handler

We also need to make a function that will react to the event and handle it.

```python
@app.on_dynamodb_record(stream_arn=stream_arn)
def handle_stream(event):
    for record in event:
        stream_data = record.new_image
        
        app.log.debug(stream_data)
```
By now our application has the permissions to read the stream and the application code to do so.
If you remember, the general idea is that this function will be triggered when a new item is added to the table. So lets
see if and how that works.

#### Deploy

We can only see the interaction between DynamoDB streams and our Lambda function when they are deployed to AWS; 
stream data can not reach our Chalice that is running locally. So, lets deploy!

As expected, the deployment failed because our function does not have the required permissions.
![](/images/code_screenshots/80_50_1.svg)

Fixing those permissions is our next task!