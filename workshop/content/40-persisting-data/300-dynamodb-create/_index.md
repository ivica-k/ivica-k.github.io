+++
title = "2.3 Infra - create our DynamoDB table"
chapter = true
weight = 300
+++

# Infrastructure caveat

The general idea of this workshop is to show the basics of serverless app development with Python. Managing AWS
infrastructure efficiently and with the correct tools is a secondary goal. That is why we will be using AWS CLI or 
simple Python scripts to provision infrastructure. Real-world projects would rely on some form of infrastructure-as-code (IaC),
using tools like:

- AWS CloudFormation
- AWS CDK (Cloud Development Kit)
- Terraform
- Pulumi
- Ansible

## Create our DynamoDB table

```bash
aws dynamodb create-table --table-name $WORKSHOP_NAME-savealife-$ENV \
  --profile workshop \
  --attribute-definitions AttributeName=first_name,AttributeType=S \
  --key-schema AttributeName=first_name,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

With this command we created a DynamoDB table with:
- The primary key being `first_name`. We talked about primary keys on the [DynamoDB 101](./200-dynamodb-101.html) page.
- The `BILLING_MODE` set to `PAY_PER_REQUEST`. This basically means that our table will scale to $0 if there is no usage.
It can also scale to $MANY if there is a lot of usage. Know your data access patterns.

Running the command from above will result in output similar to:

```bash{linenos=false}
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "first_name",
                "AttributeType": "S"
            }
        ],
        "TableName": "ivica-savealife-dev",
        "KeySchema": [
            {
                "AttributeName": "first_name",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "CREATING",
        "BillingModeSummary": {
            "BillingMode": "PAY_PER_REQUEST"
        }
    }
}
```

This step is crucial, please don't proceed further if this step failed and ask for assistance.