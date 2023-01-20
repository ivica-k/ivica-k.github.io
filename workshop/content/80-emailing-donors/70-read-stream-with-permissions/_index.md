---
title: "6.7 Reading stream data - for real"
chapter: true
weight: 70
---

# Read stream data for real

On the previous page we made sure that separate functions have separate permissions and separate
environment variables (implementing the so-called least privilege principle).

`chalice deploy` and notice how the output changes slightly:

![](/images/code_screenshots/80_70_1.png)

We can notice several new things happening:
- a new Lambda function, named `ivica-savealife-dev-handle_stream` is created
- a new IAM role was created for it (and the permissions policy was attached to the role)
- the function is subscribed to the stream we created
- permissions policy for `ivica-savealife-dev-api_handler` was updated

Looking at the AWS console, we can also notice that a trigger for our `handle_stream` function was added:

![](/images/stream_trigger.png)

From now on, whenever a new write operations happens on our table, this Lambda function will be triggered/invoked.

Create a new donation

```bash
http -b POST $(chalice url)/donation/create city=Amsterdam address="Main street" datetime="2022-04-06T12:00:00"
```

and follow the logs with `chalice logs -n handle_stream --follow`. `-n` is used to specify the name of the function whose logs
we want to view and `--follow` will poll for new log lines continuously.

The log line in my case looks like this:

![](/images/code_screenshots/80_70_2.png)

Looking at it more closely we notice that it is a bit strange; what are these `'S'` keys there...? Well, they are proof
that this event came from DynamoDB streams, since DynamoDB has data types and `'S'` stands for `String`.

Adding a donor will work in a similar fashion:

```bash
http -b POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com" type="0+" city="Haarlem"
```

and logs show

![](/images/code_screenshots/80_70_3.png)

### Amazing, right!?

Well, almost. The above messages show us that DynamoDB streams are indeed working, sending events when a new item is added
and that our function is handling those events.

What is not so great about it is that both the "Donor sign-up" and the "New blood donation event" are handled. We want
to send emails to donors from Haarlem whenever a new blood donation event is created in Haarlem. No events should be
handled by our function when a new donor signs up.

We need a way to filter these stream events and only react to those that are emitted when a new blood donation event is
created.