+++
title = "Functional requirements"
chapter = true
weight = 200
+++

# Functional requirements
Let's go over the functional requirements again. 

#### Client requested
- Web based application
- Incurs little to no cost when unused
- Does not need a big team of people to maintain it - ideally volunteers can do it

There are obvious and less obvious reasons for what they need.

## Client requirements, distilled

#### Web based
The application will be used from a web browser on a PC/laptop or a mobile device. This means that the user interface 
should be _responsive_ and _lightweight_, with an _API backend_ for handling backend tasks.

_We need an API_ to provide data to the web application.

#### Incurs little to no cost when unused
Primary users of this application will be non-profits and NGOs with _tight budgets_. Ideally they will pay for 
the application hosting only when the application is used.

_We need cloud services with the pay-as-you-go model_. Serverless offerings from AWS offer just that.

Serverless technologies allow us to **build and deploy** our applications quickly **and not worry about software patches,
security settings or capacity** because someone else takes care of all that. 

Another great benefit is the **pay-as-you-go model** - if there is no application usage your hosting costs may be 0 
euros. But, if you misjudge application usage patterns your serverless app can go from very cheap to 
_very expensive_ in a matter of minutes.

When we talk about serverless technologies, most people think of AWS Lambda, but serverless landscape
is so much more - AWS S3 is a serverless service for blob storage, SQS is a serverless messaging queue, 
Lambda is a serverless compute engine and so on.

There are servers in serverless of course. To paraphrase an AWS Serverless Hero [Slobodan Stojanovic](https://twitter.com/slobodan_/status/769582539396771844):
> Serverless is without servers in the same way that WIFI is without wires.

#### Does not need a big team of people to maintain it
Having a tight budget or volunteer workforce pushes us in the direction of using _widely accepted_ and popular tools
and services so that people capable of using them _are not hard to find_ or it is not difficult to train them.

_We need a simple yet powerful programming language_. Python is the right fit here without question.

##### Why Python?
Because this workshop was created for [PyCon Italia](https://pycon.it/en/talk/building-serverless-application-on-aws-with-python?day=2022-06-04).

Also, because Python is a first-class citizen on AWS with many tools, services, drivers and workshops available. You can do:
 - account and resource management using [Boto](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
 - infrastructure-as-code using [AWS CDK for Python](https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-python.html)
 - build and run apps on [Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-apps.html) or [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html)
 - data science with [Managed workflows for Apache Airflow](https://docs.aws.amazon.com/mwaa/latest/userguide/what-is-mwaa.html)
 - machine learning with [SageMaker Python SDK](https://github.com/aws/sagemaker-python-sdk)
