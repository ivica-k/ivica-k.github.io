+++
title = "1.3 Donor sign-up"
chapter = true
weight = 300
+++

# Donor sign-up

Up until now we've seen how a very basic AWS Lambda function looks like and how to execute it via the AWS Console. 
Our donor sign-up function will of course _not_ be triggered from the AWS Console - an HTTP POST request will trigger it.

Looking at it from the architecture standpoint, this is what we want to achieve:

![](/images/donor_signup_arch.png)

If we transformed it into steps, they would look like so:

1. A user sends an HTTP `POST` request with a JSON payload
2. API Gateway receives it and invokes the Lambda function
3. Lambda function receives the JSON payload in the `event` argument

#### API design

Our `donor_signup` Lambda function will receive the above JSON payload in the `event` argument. Let's code that!
Because our application will be API-first we should think of a good endpoint schema design - or in simple terms, how 
our API URLs will look like.

A good practice is to use [nouns to represent resources and verbs to represent actions](https://restfulapi.net/resource-naming/).
In our case, that resource is the `donor` and the action could be `signup` (or `create` etc.).

I'll go with **`/donor/signup`**.

But how do we make this happen with Chalice? It's easier than you might think :)

Replace the content in `~/serverless_workshop/savealife/app.py` with:

{{<highlight python "hl_lines=16">}}
from os import getenv
from chalice import Chalice

try:
    from dotenv import load_dotenv

    load_dotenv()
except ImportError:
    pass

first_name = getenv("WORKSHOP_NAME", "ivica")  # replace with your own name of course

app = Chalice(app_name=f"{first_name}-savealife")


@app.route("/donor/signup", methods=["POST"])
def donor_signup():
    pass

{{</highlight>}}

The most interesting line is highlighted. With it we configure two important parameters of our API:
- With `"/donor/signup"` we're telling the API Gateway that the route for our Lambda function is `/donor/signup`
- With `methods=["POST"]` we specify that this route should only react to a `POST` request

Verifying that locally shows that we were successful - only `POST` requests are accepted:

```bash{linenos=false}
http -b :8000/donor/signup
```

Output should be similar to:
```bash{linenos=false}
{
    "Code": "MethodNotAllowedError",
    "Message": "Unsupported method: GET"
}
```

```bash{linenos=false}
http -b POST :8000/donor/signup
```

But the `POST` request will work (although the endpoint returns `null` - it does not error):
```bash{linenos=false}
null
```

When it comes to the JSON payload itself I would propose we keep things simple. What we need from potential donors
are 4 pieces of information: their first name, an email for contact, their blood type and the city where they live.
So, something like this:

```json
{
  "first_name": "itsme",
  "email": "itsme@server.com",
  "blood_type": "A-",
  "city": "Amsterdam"
}
```

#### Lambda function

For now our Lambda function does nothing. Let's make it accept the JSON payload sent to it and simply return it to the caller.

{{<highlight python "hl_lines=17 19">}}
from os import getenv
from chalice import Chalice

try:
    from dotenv import load_dotenv

    load_dotenv()
except ImportError:
    pass

first_name = getenv("WORKSHOP_NAME", "ivica")  # replace with your own name of course

app = Chalice(app_name=f"{first_name}-savealife")

@app.route("/donor/signup", methods=["POST"])
def donor_signup():
    body = app.current_request.json_body

    return body
{{</highlight>}}

Chalice comes with a handy helper function for extracting the JSON body of the event sent from the API Gateway - `app.current_request.json_body`.
Next to the JSON body, it contains other useful information such as the path that was invoked, headers, the HTTP method used and others.
For the curious, [Chalice Request class](https://aws.github.io/chalice/api.html#request).

Invoking our function locally with the JSON payload from above results in:

```bash{linenos=false}
http -b POST :8000/donor/signup first_name=ivica email="ivica@server.com" city=Amsterdam type=A+
```

Output should be similar to:
```bash{linenos=false}
{
    "city": "Amsterdam",
    "email": "ivica@server.com",
    "first_name": "ivica",
    "type": "A+"
}
```

## Does it work for you?

If it does, go ahead and deploy it with `chalice deploy` and make sure it works when running on AWS.

```bash{linenos=false}
http -b POST $(chalice url)/donor/signup first_name=ivica email="ivica@server.com" city=Amsterdam type=A+
```

Output should be similar to:
```bash{linenos=false}
{
    "city": "Amsterdam",
    "email": "ivica@server.com",
    "first_name": "ivica",
    "type": "A+"
}
```

If it does **not work** for you now is the time to fix it because the follow-up pages will assume that all the code examples
up to now were working.

***
{{% notice warning %}}
The JSON payload is not checked against a model/schema - we can send anything we like:
{{% /notice %}}

```bash{linenos=false}
http -b POST $(chalice url)/donor/signup whoami=meow
```

Output should be similar to:
```bash{linenos=false}
{
    "whoami": "meow"
}
```