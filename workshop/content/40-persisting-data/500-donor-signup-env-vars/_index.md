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

![](/images/code_screenshots/40_500_1.png)

#### Table name through environment variables

One of the principles of [12-factor apps](https://12factor.net/) states that configuration should be stored
in the environment. Having configuration in the environment allows us to use the same codebase in different scenarios:
one set of configuration for the sandbox (development) account and another one for our production account.

Setting and changing environment variables in Chalice is done through the `.chalice/config.json` file.

Make the change by executing:

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

After making the changes with the command from above, your `.chalice/config.json` file looks similar to:
![](/images/code_screenshots/40_500_2.png)

**Deploy the changes**.

If everything went well the Lambda function in AWS will have the environment variable configured and the code will read it.

![](/images/donor_signup_env_vars.png)

Invoke the function again to make sure this change was applied:

```bash{linenos=false}
http -b POST $(chalice url)/donor/signup first_name=ivica
```
will return `null` because an exception was caught. Let's see what is going on with
`chalice logs`:

![](/images/code_screenshots/40_500_3.png)

Even though we still can't perform the `PutItem` operation the name of the table is now shown correctly as `ivica-savealife-dev`. A small victory, but a victory.

The following page will deal with the required permissions.