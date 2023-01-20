+++
title = "Project setup"
chapter = true
weight = 100
+++

## Project setup

For this workshop we will be using [AWS Chalice](https://aws.github.io/chalice/) - a framework for writing serverless, 
API first and/or event driven applications. Anyone familiar with Flask or FastAPI should feel right at home with the 
decorator-based approach of Chalice.

I chose it because it is simple enough to get started in minutes and yet powerful enough to give you a taste of writing 
scalable apps on AWS.
Developers like using it because it takes care of the heavy lifting for them and provisions (most of) the infrastructure 
needed.

It provides a very good developer experience since everything, the application and its infrastructure, can be
deployed right from your machine or from a CI/CD system with a single command.

#### Steps

1. Clone the example repository with: `git clone git@github.com:ivica-k/serverless-workshop-code.git`
2. Navigate to where you cloned the repository and in that folder
3. Run the setup script and fill-in the answers: `./workshop_setup.sh`
4. AWS credentials:
   1.  Classroom-like setting: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` values will be provided to you.
   2. Use the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` values of the IAM user or assumed role.
5. Make use of the `.env` file which contains **important** variables:

```bash{linenos=false}
{
    set -a
    source savealife/.env
    set +a
}
```

{{% notice warning %}}
Step 5 is needed in every terminal window you may use in this workshop. Not having the environment
variables present will make certain commands or actions fail.
{{% /notice %}}