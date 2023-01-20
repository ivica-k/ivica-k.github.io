---
title: "6.4 DynamoDB streams 101"
chapter: true
weight: 40
---

# DynamoDB streams 101

![](/images/email_donors_streams.png)

## Streams 101

It is possible to configure your DynamoDB table to emit events when item-level changes occur - this is called a "stream".
Streams are used to enable post-processing on the data that was written to the table.
When enabled, you can configure a stream and choose the information that will be written to it whenever the
data in the table is modified:

- `KEYS_ONLY` — Only the key attributes of the modified item.
- `NEW_IMAGE` — The entire item, as it appears after it was modified.
- `OLD_IMAGE` — The entire item, as it appeared before it was modified.
- `NEW_AND_OLD_IMAGES` — Both the new and the old images of the item.

Every stream event that is emitted consists of stream records. Every stream record is a single modification to the 
DynamoDB table data. In practice, if seven `PutItem` operations are executed on the table it would create seven stream 
records.

#### Create the stream

In this case we will use `NEW_IMAGE` because it will only hold information about the item that was inserted just now. This
AWS CLI command can be used to update our existing table and enable the stream:

```bash
aws dynamodb update-table \
  --table-name $WORKSHOP_NAME-savealife-$ENV \
  --stream-specification StreamEnabled=true,StreamViewType="NEW_IMAGE"
```

The output will contain the stream ARN (Amazon Resource Name, which uniquely identifies AWS resources):

![](/images/code_screenshots/80_40_1.png)

Append the value of `LatestStreamArn` to the `.env` file, we will need it for later:

![](/images/code_screenshots/80_40_2.png)
