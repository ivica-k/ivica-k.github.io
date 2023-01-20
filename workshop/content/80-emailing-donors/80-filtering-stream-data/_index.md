---
title: "6.8 Filtering stream data"
chapter: true
weight: 80
---

# Filtering stream data

On the previous page we saw how DynamoDB streams are being handled by our `handle_stream` function a bit too well :)
The "Donor sign-up" and the "New blood donation event" were handled. We want to send emails to donors from Haarlem 
whenever a new blood donation event is created in Haarlem. No events should be handled by our function when a new donor
signs up.

![](/images/email_donors_streams.png)

## One way to do it...

One way to filter out events we don't want is to implement a few lines of code that will simply discard the events
that we're not interested in and react to events that we are interested in.

The "donation" event looks like:

![](/images/code_screenshots/80_80_1.png)

and the "donor sign-up" event looks like:

![](/images/code_screenshots/80_80_2.png)

It would be fairly easy to check whether the `PK` starts with a `DONOR#` or `DONATION#`:

![](/images/code_screenshots/80_80_3.png)

But filtering in this way also introduces several issues:
- adds complexity in the business logic portion of the application. Code is a liability that must be tested and maintained
- incurs unnecessary cost; our `handle_stream` function is invoked whenever an item is inserted into the table, even if 
that item will be discarded
- the ratio of donor sign-ups compared to blood donation events can be very uneven; hundreds of donors can sign up in a 
day, while typically only a couple blood donation events will be scheduled per week

There must be a better way to filter out unwanted events.

## A better way to do it - Lambda event filtering

[Lambda event filtering](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventfiltering.html) allows us to control
which events are to be processed. In our example, we could create a filter that takes into account only those events 
that have a `PK` value that starts with the string `DONATION`. Event filtering is done on the AWS side so it does not 
incur any costs for us; quite the opposite.

These filters can also be tested easily because they share the syntax and rules with the 
[AWS EventBridge](https://aws.amazon.com/eventbridge/) event patterns.

Filters are defined through fairly simple JSON objects and AWS provides several built-in [comparison operators](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventfiltering.html#filtering-syntax), such as:
- `prefix` which checks whether a value starts with a string; `"pk": [ {"prefix": "DONATION" } ]`
- `null` which checks whether a value is null; `"city": [ null ]`
- `equals` which checks whether a value is equal to a value; `"city": [ "Haarlem" ]`

One downside of event filtering is that Chalice as a tool does not support them natively; we can still create and use
them, but they have to be created outside of Chalice.

#### Creating an event filter

Lambda functions and filters are connected together through a feature called "[event source mappings](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventsourcemapping.html)".
An event source mapping is a resource that reads from an event source (DynamoDB stream, AWS SQS, Apache Kafka...) and 
invokes a Lambda function.

You may think of it as a pair made up of `EVENT_SOURCE:LAMBDA_FUNCTION`.

This event source mapping is what sets the trigger for a Lambda function, and if you remember we already created one
of those (Chalice did it for us):

![](/images/stream_trigger.png)

But because Chalice does not (yet) support filters for event sources we have to do it ourselves. No biggie.
The filter itself looks like this:

![](/images/code_screenshots/80_80_4.png)

and can be roughly translated into 
![](/images/code_screenshots/80_80_5.png)

##### Get the ID of the existing mapping to include the filter

```bash
MAP_UUID=$(aws lambda list-event-source-mappings --function-name $WORKSHOP_NAME-savealife-$ENV-handle_stream | jq -r .EventSourceMappings[0].UUID)
```

##### Update the existing event source mapping

```bash
aws lambda update-event-source-mapping \
  --uuid $MAP_UUID \
  --filter-criteria 'Filters=[{Pattern="{\"dynamodb\": {\"NewImage\": {\"PK\": {\"S\": [{\"prefix\": \"DONATION\"}]}}}}"}]'
```

which results in a filter added to our trigger

![](/images/stream_trigger_filter.png)

## Does it work?

- deploy; `chalice deploy`
- watch the logs; `chalice logs -n handle_stream --follow`
- create a new donor; `http -b POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com" type="0+" city="Haarlem"`
- create a new donation event; `http -b POST $(chalice url)/donation/create city=Amsterdam address="Main street" datetime="2022-04-06T12:00:00"`

The logs will show the following:

![](/images/code_screenshots/80_80_6.png)

Notice how there are no logs for the donor sign-up that we performed just now? That's the filters at work; the Lambda function
is not being invoked for donor sign-ups, it is invoked only for blood donation events.

Create a few more donors just to be sure (while following the logs). Those events will be ignored :thumbsup:

![](/images/0_damage.jpeg)