+++
title = "2.6 Permissions error"
chapter = true
weight = 600
+++

# Persist to DynamoDB - permissions error

On the previous page we saw that invoking our function returns an error that is permission related.

```bash{linenos=false}
http -b POST $(chalice url)/donor/signup first_name=ivica
```
will output lines similar to:

![](/images/code_screenshots/40_600_1.png)

On this page we will
learn how to adjust permissions that our AWS Lambda functions has.

#### Changing Lambda function permissions

So far we've seen that Chalice was able to automatically generate AWS resources for us, including IAM roles and permissions.

_Why doesn't it do so in this case?_

If you guessed "because the DynamoDB table was created outside of Chalice" you guessed correctly. Chalice is not managing
the table - it has no idea it even exists and because of that it can not generate the required permission policies. We 
can though :)

Create the `.chalice/policy-dev.json` file with the following contents:

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
        "arn:aws:dynamodb:*:*:table/$WORKSHOP_NAME-savealife-dev"
      ],
      "Effect": "Allow"
    }
  ]
}
EOF
```

After making the changes with the command from above, your `.chalice/policy-dev.json` file looks similar to:
![](/images/code_screenshots/40_600_2.png)

Looking closely into it we can discern two statements:
- The first statement allows our Lambda function to perform actions on logs
- The second statement allows our Lambda function to perform actions/queries on our table

The next step for us is to tell Chalice to not automatically generate the permissions policy because we are supplying
our own, and that can be easily done in the `.chalice/config.json` file:

{{<highlight bash "hl_lines=8">}}
cat > .chalice/config.json <<EOF
{
  "version": "2.0",
  "app_name": "$WORKSHOP_NAME-savealife",
  "stages": {
    "dev": {
      "api_gateway_stage": "api",
      "autogen_policy": false,
      "environment_variables": {
        "TABLE_NAME": "$WORKSHOP_NAME-savealife-dev"
      }
    }
  }
}
EOF
{{</highlight>}}
