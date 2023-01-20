+++
title = "Intro"
chapter = true
weight = 10
+++

# Ways to follow

To complete this workshop, an AWS account is needed which will host the resources you create.

## Classroom-like setting

AWS account, necessary credentials and required permissions will be provided to you. All cost incurred while using the
resources from this workshop are associated to the AWS account provided to you.

One or more _mentors_ are available to you - **make use** of their presence! Ask questions, look for guidance, 
explanations and similar.

## On your own

If you plan to follow this workshop on your own, you need to use a personal account, or create a new AWS account, making
sure that you have the necessary permissions for AWS services used in this workshop. All cost incurred while using the
resources from this workshop are at your own expense.

Permissions policy required for this workshop is:

{{%expand "Click here for the policy" %}}
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "dynamodb:TagResource",
                "dynamodb:PutItem",
                "dynamodb:Scan",
                "dynamodb:Query",
                "dynamodb:DeleteTable",
                "dynamodb:UpdateTable"
            ],
            "Resource": [
				"arn:aws:dynamodb:*:AWS_ACCOUNT_ID:table/*",
				"arn:aws:dynamodb:AWS_REGION:AWS_ACCOUNT_ID:table/*-savealife-dev"
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetShardIterator",
                "dynamodb:DescribeStream",
                "dynamodb:ListStreams",
                "dynamodb:GetRecords"
            ],
            "Resource": [
                "arn:aws:dynamodb:eu-west-1:AWS_ACCOUNT_ID:table/*-savealife-dev/stream/*"
            ]
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": [
                "apigateway:POST",
                "apigateway:GET"
            ],
            "Resource": [
				"arn:aws:apigateway:AWS_REGION::/tags/*",
                "arn:aws:apigateway:AWS_REGION::/restapis",
				"arn:aws:apigateway:AWS_REGION::/restapis/*",
				"arn:aws:apigateway:AWS_REGION::/domainnames",
				"arn:aws:apigateway:AWS_REGION::/domainnames/*",
				"arn:aws:apigateway:AWS_REGION::/domainnames/*/*",
                "arn:aws:apigateway:AWS_REGION::/restapis/*/resources",
                "arn:aws:apigateway:AWS_REGION::/restapis/*/resources/*",
				"arn:aws:apigateway:AWS_REGION::/restapis/*/deployments"
			]
        },
        {
            "Sid": "VisualEditor8",
            "Effect": "Allow",
            "Action": "apigateway:DELETE",
            "Resource": [
                "arn:aws:apigateway:AWS_REGION::/restapis/*",
				"arn:aws:apigateway:AWS_REGION::/restapis/*/resources/*"
            ]
        },
        {
            "Sid": "VisualEditor16",
            "Effect": "Allow",
            "Action": "apigateway:PUT",
            "Resource": [
				"arn:aws:apigateway:AWS_REGION::/restapis/*",
				"arn:aws:apigateway:AWS_REGION::/restapis/*/methods/GET",
				"arn:aws:apigateway:AWS_REGION::/restapis/*/methods/GET/*",
				"arn:aws:apigateway:AWS_REGION::/restapis/*/methods/POST",
				"arn:aws:apigateway:AWS_REGION::/restapis/*/methods/POST/*",
				"arn:aws:apigateway:AWS_REGION::/restapis/*/methods/PUT",
				"arn:aws:apigateway:AWS_REGION::/restapis/*/methods/PUT/*"
            ]
        },
        {
            "Sid": "VisualEditor18",
            "Effect": "Allow",
            "Action": "apigateway:PATCH",
            "Resource": [
				"arn:aws:apigateway:AWS_REGION::/restapis/*",
				"arn:aws:apigateway:AWS_REGION::/domainnames/*"
			]
        },
        {
            "Sid": "VisualEditor21",
            "Effect": "Allow",
            "Action": [
                "iam:GetRole",
                "logs:*",
                "iam:UpdateRoleDescription",
                "iam:ListRoles",
                "iam:DeleteRole",
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:PutRolePolicy",
                "dynamodb:ListStreams",
                "dynamodb:CreateTable",
                "iam:GetServiceLinkedRoleDeletionStatus",
                "iam:PassRole",
                "iam:CreateServiceLinkedRole",
                "iam:DetachRolePolicy",
                "iam:DeleteRolePolicy",
                "lambda:*",
                "ec2:*",
                "iam:DeleteServiceLinkedRole",
                "iam:ListRolePolicies"
            ],
            "Resource": "*"
        }
    ]
}
```
{{% /expand%}}

Make sure to replace the `AWS_REGION` with the name of the AWS region you use and `AWS_ACCOUNT_ID` with your AWS
account ID. The permisions policy may be attached to an IAM user or an IAM role, which is completely up to you dear reader.

# Conventions

#### Code
Code, either Python or CLI commands, that you can copy/paste looks like:

```bash
echo "hello!"
```

```python
print("hello!")
```

All code samples are available in repository branches (we'll get to that soon) or on these pages. However, you are 
**strongly** encouraged to type them yourself and get a feel for the framework and supporting tools.

#### CLI output

CLI commands that you run will show some output. This output is not to be copy/pasted; it is the expected result of a
command execution. It looks like:

![](/images/code_screenshots/30_100_1.png)

#### Notes

When needed, colored sections will point out further reading material, potential pitfalls and misunderstandings. They
come in green, orange and red:

{{% notice tip %}}
This is a tip.
{{% /notice %}}

{{% notice info %}}
This is an information.
{{% /notice %}}

{{% notice warning %}}
This is a warning.
{{% /notice %}}

***

This intro chapter will explain the requirements and how those inspired our design choices and AWS services we will use 
to build the application.