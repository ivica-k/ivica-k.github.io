---
title: "6.6 Read stream permissions"
chapter: true
weight: 60
---

# Read stream permissions

The task of our `handle_stream` function is to read the stream data from the stream, 
and it should be permitted to do only that. One of the best practices when dealing with permissions is to only allow what
is absolutely needed; no more, no less. This is called the principle of least privilege.

So how can we implement the principle of least privilege with our functions?

Chalice allows us to specify [certain configuration settings](https://aws.github.io/chalice/topics/configfile.html) per 
stage or per function. Without going too much into what stages are, specifying permissions per function is supported and
is pretty straight forward.

The results we want to achieve are:
- all Lambda functions have the `TABLE_NAME` environment variable
- `handle_stream` function has the `STREAM_ARN` environment variable
- all Lambda functions have permissions to log to Cloudwatch
- `handle_stream` function has the permissions to read stream data

We can achieve these results by splitting permissions into several files:
- `policy-dev.json` will contain Cloudwatch and DynamoDB permissions
- `policy-handle-stream.json` will contain Cloudwatch and stream permissions

#### Create the policies

`policy-dev.json`

```bash
cat > .chalice/policy-dev.json <<EOF 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*",
      "Effect": "Allow"
    },
    {
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:DeleteItem",
        "dynamodb:UpdateItem",
        "dynamodb:GetItem",
        "dynamodb:Scan",
        "dynamodb:Query"
      ],
      "Resource": [
        "arn:aws:dynamodb:*:*:table/$WORKSHOP_NAME-savealife-$ENV"
      ],
      "Effect": "Allow"
    }
  ]
}
EOF
```

`policy-handle-stream.json`

```bash
cat > .chalice/policy-handle-stream.json <<EOF 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*",
      "Effect": "Allow"
    },
    {
      "Action": [
        "dynamodb:DescribeStream",
        "dynamodb:GetRecords",
        "dynamodb:GetShardIterator",
        "dynamodb:ListStreams"
      ],
      "Resource": [
        "arn:aws:dynamodb:eu-central-1:932785857088:table/$WORKSHOP_NAME-savealife-$ENV/stream/*"
      ],
      "Effect": "Allow"
    }
  ]
}
EOF
```

#### Policy and environment variables per function

The config file `.chalice/config.json` can be edited to assign policies and environment variables where they are needed:

```bash
cat > .chalice/config.json <<EOF 
{
  "version": "2.0",
  "app_name": "$WORKSHOP_NAME-savealife",
  "autogen_policy": false,
  "automatic_layer": true,
  "stages": {
    "dev": {
      "api_gateway_stage": "api",
      "iam_policy_file": "policy-dev.json",
      "environment_variables": {
        "TABLE_NAME": "$WORKSHOP_NAME-savealife-$ENV"
      },
      "lambda_functions": {
        "handle_stream": {
          "iam_policy_file": "policy-handle-stream.json",
          "environment_variables": {
            "STREAM_ARN": "STREAM_ARN_HERE"
          }
        }
      }
    }
  }
}
EOF
```

Don't forget to replace the `STREAM_ARN_HERE` with your actual stream ARN.
