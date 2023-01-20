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

One potential solution using SES would look something like this:
- use DynamoDB streams to capture any newly created blood donation events
- when a new blood donation event is created, we fetch all the donors from the city where the donation event will happen
- batch up to 50 donor's emails and send and invoke SES `send_email()` function

![](/images/email_donors_streams.png)

Before going any deeper into DynamoDB streams, we will focus on the second point from our list above: 
"when a new blood donation event is created, we fetch all the donors from the city where the donation event will happen".

<!-- **I choose Simple Email Service (SES)** with DynamoDB streams because: -->
<!-- - it offers more emails for free every month and allows us to send pretty emails. -->
<!-- - it is technically more interesting to develop -->