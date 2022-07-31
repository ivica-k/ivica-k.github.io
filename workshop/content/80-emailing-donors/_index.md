---
title: "6.0 Emailing donors"
chapter: true
weight: 80
---

# Emailing donors

If you recall, our application will have several features, and so far we built 2 out of 3. Now is the time to build the
last feature:

- A person can sign-up to become a donor
- An organization can create blood donation events
- **Donors receive an email when a new event is created in their city**

## Sending emails on AWS

There are multiple ways of sending emails on AWS. Each has their own benefits and drawbacks, and we should consider both
when choosing a service to send emails with.

Two main options that come to mind are:
- [AWS Simple Email Service (AWS SES)](https://aws.amazon.com/ses/)
- [AWS Simple Notification Service (AWS SNS)](https://aws.amazon.com/sns/).

AWS SES is a reliable and scalable SMTP system. It fits our use case because it supports the pay-as-you-go model. When
it comes to advanced features, SES supports HTML emails (with graphics, tables and similar), email templates and a
reporting system for bounces, deliveries, and complaints; all three of which are very important for your email 
sending reputation. As for the drawbacks, SES can email up to 50 recipients at a time so a batching logic must be built 
for it.

Cost? 62.000 emails free per month, $10 for 100.000 emails after that.

[//]: # (AWS SNS is not a system for sending emails - it is a notification system. It so happens that one of the notification)

[//]: # (mechanisms is to send an email. It was built to support massive scale and the pricing is my favourite: pay-as-you-go model.)

[//]: # (Contrary to SES, SNS will not send emails to addresses that are not _subscribed_. That means that every donor we register)

[//]: # (would receive an email, asking if they would like to receive further notifications from us. Neat.)

[//]: # (When it comes to drawbacks, SNS supports only plain-text emails &#40;without images, CSS etc.&#41; but you can include HTML)

[//]: # (links in them.)

[//]: # (## Making a choice)

[//]: # ()
[//]: # (Benefits and drawbacks are pretty clear when it comes to SES and SNS. I would also like to do a technical evaluation)

[//]: # (of both to reach a more clear conclusion.)

[//]: # (#### SES)

[//]: # (SES will happily send pretty emails for us, but only to max 50 recipients at a time, meaning that we need to)

[//]: # (implement that logic somewhere. And how do we know when to send emails, what is the trigger?)

One potential solution using SES would look something like this:
- use DynamoDB streams to capture any newly created blood donation events
- when a new blood donation event is created, we fetch all the donors from the city where the donation event will happen
- batch up to 50 donor's emails and send and invoke SES `send_email()` function

![](/images/email_donors_streams.png)

Before going any deeper into DynamoDB streams, we will focus on the second point from our list above: 
"when a new blood donation event is created, we fetch all the donors from the city where the donation event will happen".

[//]: # (- stash donor emails in [AWS Simple Queue Service &#40;AWS SQS&#41;]&#40;https://aws.amazon.com/sqs/&#41;. One million requests per month are free)
[//]: # (- trigger a Lambda function from SQS that will interact with AWS SES and send emails)

[//]: # (- use those details to invoke SES with 50 addresses at a time until there are no more details. AWS Lambda fits here well)

[//]: # (SES, being the _Simple_ Email Service it is, does not offer too many integrations. Two main ways of interacting with it)
[//]: # (are good ol' SMTP and the SES API &#40;through an SDK&#41;.)



[//]: # (#### SNS)

[//]: # ()
[//]: # (Using SNS to send notifications requires a topic &#40;a communication channel&#41; and subscribers to subscribe to that topic.)

[//]: # (In our use case we can potentially do the following:)

[//]: # ()
[//]: # (- whenever a new donation event is created, we create a topic for the city where the donation event will happen)

[//]: # (- when a new donor signs up, we subscribe them to the topic of the city where they live)

[//]: # (- SNS sends them a question whether they want to receive further notifications)

[//]: # (- whenever a new donation event is created, we send a message &#40;through an SDK&#41; to the corresponding topic)

[//]: # (- SNS notifies everyone that is subscribed &#40;and confirmed&#41; to the topic with a simple email)

[//]: # ()
[//]: # (Cost? 1000 notifications free, $2 per 100.000 notifications.)

<!-- **I choose Simple Email Service (SES)** with DynamoDB streams because: -->
<!-- - it offers more emails for free every month and allows us to send pretty emails. -->
<!-- - it is technically more interesting to develop -->

[//]: # (dodaj neke grafike, objasni ddb streams)