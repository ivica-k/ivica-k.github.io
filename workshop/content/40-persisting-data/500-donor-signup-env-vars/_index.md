+++
title = "2.5 Table name error"
chapter = true
weight = 500
+++

# Persist to DynamoDB - table name error

On the previous page we saw that invoking our function locally returns an error

```bash{linenos=false}
http -b POST $(chalice url)/donor/signup first_name=ivica
```

will error with:

```bash
ivica-savealife - ERROR - Caught exception for path /donor/signup
Traceback (most recent call last):
  File "pythonlib/python3.10/site-packages/chalice/app.py", line 1752, in _get_view_function_response
    response = view_function(**function_args)
  ... SNIP ...
  File "pythonlib/python3.10/site-packages/boto3/resources/base.py", line 125, in __init__
    raise ValueError(f'Required parameter {identifier} not set')
ValueError: Required parameter name not set
```

[//]: # (```bash{linenos=false})

[//]: # (An error occurred &#40;AccessDeniedException&#41; when calling the PutItem operation: )

[//]: # (User: arn:aws:sts::932785857088:assumed-role/ivica-savealife-dev/ivica-savealife-dev is not authorized to perform: )

[//]: # (dynamodb:PutItem on resource: arn:aws:dynamodb:eu-central-1:932785857088:table/None-savealife-dev)

[//]: # (```)

[//]: # ()
[//]: # (The message is very detailed and it is indicative of at least two problems:)

[//]: # (- The first problem is the `AccessDeniedException`: our Lambda function does not have permissions to perform the `PutItem` )

[//]: # (operation &#40;this page&#41;)

[//]: # (- The second problem is the name of our table `None-savealife-dev` which comes from the fact that our Lambda function)

[//]: # (does not have access to the `.env` file &#40;as it is supposed to be&#41; and we have to handle environment variables for AWS differently)

[//]: # (  &#40;next page&#41;)


#### Table name through environment variables

One of the principles of [12-factor apps](https://12factor.net/) states that configuration should be stored
in the environment. Having configuration in the environment allows us to use the same codebase in different scenarios:
one set of configuration for the sandbox (development) account and another one for our production account.

Setting and changing environment variables in Chalice is done through the `.chalice/config.json` file.
Make the change by executing:

[//]: # (Environment variables can be specified per stage)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # (cat .chalice/config.json)

[//]: # (```)

[//]: # (like so:)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # ({)

[//]: # (  "version": "2.0",)

[//]: # (  "app_name": "ivica-savealife",)

[//]: # (  "stages": {)

[//]: # (    "dev": {)

[//]: # (      "api_gateway_stage": "api",)

[//]: # (      "environment_variables": {)

[//]: # (        "TABLE_NAME": "dev-table",)

[//]: # (        "OTHER_CONFIG": "dev-value")

[//]: # (      })

[//]: # (    })

[//]: # (  })

[//]: # (})

[//]: # (```)

[//]: # ()
[//]: # (We could even do a more granular approach where environment variables are specified per function)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # (cat .chalice/config.json)

[//]: # (```)

[//]: # (like so:)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # ({)

[//]: # (  "version": "2.0",)

[//]: # (  "app_name": "ivica-savealife",)

[//]: # (  "stages": {)

[//]: # (    "dev": {)

[//]: # (      "api_gateway_stage": "api",)

[//]: # (      "lambda_functions": {)

[//]: # (        "donor_signup": {)

[//]: # (          "environment_variables": {)

[//]: # (            "TABLE_NAME": "dev-table",)

[//]: # (            "OTHER_CONFIG": "dev-value")

[//]: # (          })

[//]: # (        })

[//]: # (      })

[//]: # (    })

[//]: # (  })

[//]: # (})

[//]: # (```)

[//]: # (##### Make the change)

[//]: # (Set the contents of `.chalice/config.json` with:)

```bash{linenos=false}
cat > .chalice/config.json <<EOF
{
  "version": "2.0",
  "app_name": "$WORKSHOP_NAME-savealife",
  "stages": {
    "dev": {
      "api_gateway_stage": "api",
      "environment_variables": {
        "TABLE_NAME": "$WORKSHOP_NAME-savealife-dev"
      }
    }
  }
}
EOF
```

[//]: # (Our code in `db.py` now needs to read the `TABLE_NAME` from the environment:)

[//]: # ()
[//]: # ({{<highlight python "hl_lines=15">}})

[//]: # (import logging)

[//]: # (from os import getenv)

[//]: # ()
[//]: # (import boto3)

[//]: # (from dotenv import load_dotenv)

[//]: # ()
[//]: # (load_dotenv&#40;&#41;)

[//]: # ()
[//]: # (ENV = getenv&#40;"ENV", "dev"&#41;)

[//]: # (first_name = getenv&#40;"WORKSHOP_NAME", "ivica"&#41;  # replace with your own name of course)

[//]: # ()
[//]: # (logger = logging.getLogger&#40;f"{first_name}-savealife"&#41;)

[//]: # ()
[//]: # (_DB = None)

[//]: # (TABLE_NAME = getenv&#40;"TABLE_NAME"&#41;)

[//]: # ()
[//]: # ()
[//]: # (def get_app_db&#40;&#41;:)

[//]: # (    global _DB)

[//]: # ()
[//]: # (    if _DB is None:)

[//]: # (        _DB = SavealifeDB&#40;)

[//]: # (            table=boto3.resource&#40;"dynamodb"&#41;.Table&#40;TABLE_NAME&#41;,)

[//]: # (            logger=logger)

[//]: # (        &#41;)

[//]: # ()
[//]: # (    return _DB)

[//]: # ()
[//]: # ()
[//]: # (class SavealifeDB:)

[//]: # (    def __init__&#40;self, table, logger&#41;:)

[//]: # (        self._table = table)

[//]: # (        self._logger = logger)

[//]: # ()
[//]: # (    def donor_signup&#40;self, donor_dict&#41;:)

[//]: # (        try:)

[//]: # (            self._table.put_item&#40;)

[//]: # (                Item={)

[//]: # (                    "first_name": donor_dict.get&#40;"first_name"&#41;,)

[//]: # (                    "city": donor_dict.get&#40;"city"&#41;,)

[//]: # (                    "type": donor_dict.get&#40;"type"&#41;,)

[//]: # (                    "email": donor_dict.get&#40;"email"&#41;,)

[//]: # (                })

[//]: # (            &#41;)

[//]: # (            self._logger.debug&#40;f"Inserted donor '{donor_dict.get&#40;'email'&#41;}' into DynamoDB table '{self._table}'"&#41;)

[//]: # ()
[//]: # (            return True)

[//]: # ()
[//]: # (        except Exception as exc:)

[//]: # (            self._logger.exception&#40;exc&#41;)

[//]: # ()
[//]: # (            return False)

[//]: # ()
[//]: # ({{</highlight>}})

**Deploy the changes**.

If everything went well the Lambda function in AWS will have the environment variable configured and the code will read it.

![](/images/donor_signup_env_vars.png)

Invoke the function again to make sure this change was applied:

```bash{linenos=false}
http -b POST $(chalice url)/donor/signup first_name=ivica
```
will return `null` because an exception was caught. Let's see what is going on with
`chalice logs`:

```bash{linenos=false}
An error occurred (AccessDeniedException) when calling the PutItem operation: 
User: arn:aws:sts::932785857088:assumed-role/ivica-savealife-dev/ivica-savealife-dev is not authorized to perform: 
dynamodb:PutItem on resource: arn:aws:dynamodb:eu-central-1:932785857088:table/ivica-savealife-dev
```

Even though we still can't perform the `PutItem` operation the name of the table is now shown correctly as `ivica-savealife-dev`. A small victory, but a victory.

The following page will deal with the required permissions.