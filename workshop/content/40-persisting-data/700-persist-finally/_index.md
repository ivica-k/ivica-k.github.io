+++
title = "2.7 Persist the donor to DynamoDB - for real"
chapter = true
weight = 700
+++

# Persist the donor to DynamoDB - for real

On the previous page we dealt with permission errors which were too strict and our Lambda function did not have permissions
to write data to the DynamoDB table. With that fixed it is finally time to persist the donor to the table.

**Deploy the changes**.

If everything went well the output of `chalice deploy` will be slightly different from previous executions

![](/images/code_screenshots/40_700_1.png)

A new IAM role named `ivica-savealife-dev-api_handler` was created, our Lambda function was updated to use it and
the old IAM role named `ivica-savealife-dev` was deleted in the end.

Invoke the function again to make sure this change was applied:

```bash{linenos=false}
http -b POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com"
```
will work :thumbsup:

Looking at the logs with `chalice logs` we can see something along the lines of:

![](/images/code_screenshots/40_700_2.png)

**Congratulations!** :thumbsup:

The function seems to be working, and it is writing data to our DynamoDB table.
![](/images/donor_signup_dynamo.png)

You may add additional items by invoking the function with a different `first_name` value. Since DynamoDB tables are 
not enforcing a schema we can also include other relevant fields.

```bash{linenos=false}
http -b POST $(chalice url)/donor/signup first_name=joe
http -b POST $(chalice url)/donor/signup first_name=candy city=Amsterdam type="A+" email="candy@server.com"
true
```

![](/images/donor_signup_dynamo_additional.png)
