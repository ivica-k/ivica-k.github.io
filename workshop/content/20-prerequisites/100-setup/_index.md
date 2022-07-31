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

[//]: # (Run the following script and fill-in the answers: )

[//]: # (```bash)
[//]: # (bash -c "$&#40;curl -s https://gist.githubusercontent.com/ivica-k/e3391169e8749a50608ba78b969da6a3/raw/workshop_setup.sh&#41;")
[//]: # (```)

`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` values will be provided to you.

[//]: # (##### Script output)

[//]: # ()
[//]: # (```bash)

[//]: # (What is your first name? ivica)

[//]: # (Enter AWS_ACCESS_KEY_ID value &#40;will not be echoed&#41;: )

[//]: # (Enter AWS_SECRET_ACCESS_KEY value &#40;will not be echoed&#41;: )

[//]: # (Environment verification:)

[//]: # (------------------------------)

[//]: # (Command output:)

[//]: # (WORKSHOP_NAME=ivica)

[//]: # (AWS_PROFILE=workshop)

[//]: # (AWS_DEFAULT_REGION=eu-central-1)

[//]: # (ENV=dev)

[//]: # ()
[//]: # (Expected output:)

[//]: # (WORKSHOP_NAME=YOUR_FIRST_NAME_HERE)

[//]: # (AWS_PROFILE=workshop)

[//]: # (AWS_DEFAULT_REGION=eu-central-1)

[//]: # (ENV=dev)

[//]: # ()
[//]: # (If the command output is not equal to the expected output, something went wrong. Ask for help :&#41;)

[//]: # ()
[//]: # (AWS credentials verification:)

[//]: # (------------------------------)

[//]: # (Command output:)

[//]: # ({)

[//]: # (    "UserId": "AROA5SLS6UJAFYZYO3O3W:botocore-session-1665226322",)

[//]: # (    "Account": "932785857088",)

[//]: # (    "Arn": "arn:aws:sts::932785857088:assumed-role/serverless_workshop_role/botocore-session-1665226322")

[//]: # (})

[//]: # ()
[//]: # (Expected output:)

[//]: # ({)

[//]: # (    UserId: AROA5SLS6UJAFYZYO3O3W:botocore-session-1234567890,)

[//]: # (    Account: 932785857088,)

[//]: # (    Arn: arn:aws:sts::932785857088:assumed-role/serverless_workshop_role/botocore-session-1234567890)

[//]: # (})

[//]: # ()
[//]: # (If the command output is not similar &#40;they can't be equal&#41; to the expected output, something went wrong. Ask for help :&#41;)

[//]: # (```)

[//]: # ()
[//]: # (As the script says, if the command output is significantly different from the values shown above, please ask for assistance.)

[//]: # (***)

[//]: # ()
[//]: # (If the script executed successfully **you can jump** to the [Donor sign-up page]&#40;../30-donor-signup.html&#41;)


[//]: # (## Manual setup)

[//]: # ()
[//]: # (Jump to the [Donor sign-up page]&#40;../30-donor-signup.html&#41; if you ran the automated setup script.)

[//]: # ()
[//]: # (#### Install Chalice)

[//]: # ()
[//]: # (Python 3.6+ recommended, up to and including 3.9.)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # ({)

[//]: # (    mkdir ~/serverless_workshop && cd ~/serverless_workshop)

[//]: # (    python -m pip install virtualenv)

[//]: # (    python -m virtualenv venv)

[//]: # (    source venv/bin/activate)

[//]: # (    python -m pip install chalice==1.27.0)

[//]: # (})

[//]: # (```)

[//]: # ()
[//]: # ({{% notice info %}})

[//]: # (MacOS users might need to create a symlink to the `python3` executable: `ln -s /usr/local/bin/python3 /usr/local/bin/python`)

[//]: # ({{% /notice %}})

[//]: # ()
[//]: # (We have the `chalice` binary now and it's version should be:)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # (chalice --version)

[//]: # (```)

[//]: # ()
[//]: # (Outputs:)

[//]: # (```bash{linenos=false})

[//]: # (chalice 1.27.0, python 3.10.2, linux 5.16.15-arch1-1)

[//]: # (```)

[//]: # ()
[//]: # (#### Create a project)

[//]: # ()
[//]: # (Our Chalice project will be called `savealife`.)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # (chalice new-project savealife)

[//]: # (```)

[//]: # ()
[//]: # (If you get a warning about using a newer Python version than supported on AWS Lambda please ignore it.)

[//]: # ()
[//]: # (#### Explore the `savealife` folder)

[//]: # ()
[//]: # (Looking at the `savealife` folder we can observe the basic Chalice skeleton:)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # (├── app.py)

[//]: # (├── .chalice)

[//]: # (    └── config.json)

[//]: # (├── .gitignore)

[//]: # (└── requirements.txt)

[//]: # (```)

[//]: # ()
[//]: # (As you can probably guess yourself, the dependencies of your app should be listed in `requirements.txt`, your Python)

[//]: # (code goes to `app.py` and `.chalice/config.json` is used for various configuration options supported by Chalice.)

[//]: # ()
[//]: # (#### Install requirements)

[//]: # ()
[//]: # (Change directory to `~/serverless_workshop/savealife/`, this is how you can create the basic requirements file,)

[//]: # (after which you can install it the usual way:)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # (cat > requirements.txt <<EOF)

[//]: # (boto3==1.24.75)

[//]: # (botocore>=1.25.2)

[//]: # (EOF)

[//]: # (```)

[//]: # ()
[//]: # (#### Configure .env)

[//]: # ()
[//]: # (Set a variable for your first name since we'll be using that one quite often. Use your own name :&#41;)

[//]: # ()
[//]: # (```bash)

[//]: # (export MY_FIRST_NAME="ivica")

[//]: # (```)

[//]: # ()
[//]: # (To make use of environment variables needed for development we will be using the [python-dotenv]&#40;https://pypi.org/project/python-dotenv/&#41;)

[//]: # (project. In short, it allows us to define arbitrary environment variables through the `.env` file, like so:)

[//]: # ()
[//]: # (```bash{linenos=false})

[//]: # (cat > .env <<EOF)

[//]: # (WORKSHOP_NAME="$MY_FIRST_NAME")

[//]: # (ENV=dev)

[//]: # (AWS_PROFILE=workshop)

[//]: # (AWS_DEFAULT_REGION=eu-central-1)

[//]: # (EOF)

[//]: # (```)

[//]: # ()
[//]: # (Please create this file in the root of your project &#40;`~/serverless_workshop/savealife/.env`&#41; and set the variables as)

[//]: # (shown above, _replacing_ your first name of course.)

4. Use the `.env` file which contains **important** variables:

```bash{linenos=false}
{
    set -a
    source savealife/.env
    set +a
}
```

[//]: # (Verify that environment variables are set correctly:)

[//]: # ()
[//]: # (```bash)

[//]: # (env | grep -ie '\&#40;^WORK\|^ENV\|^AWS\&#41;')

[//]: # (```)

[//]: # ()
[//]: # (```bash)

[//]: # (WORKSHOP_NAME=ivica)

[//]: # (AWS_PROFILE=workshop)

[//]: # (AWS_DEFAULT_REGION=eu-central-1)

[//]: # (ENV=dev)

[//]: # (```)

[//]: # ()
[//]: # ()
{{% notice warning %}}
This step is needed in every terminal window/application you may use in this workshop. Not having the environment
variables present will make certain commands/actions fail.
{{% /notice %}}

[//]: # ()
[//]: # (#### Configure Chalice with your first name)

[//]: # ()
[//]: # (You may be sharing an AWS account with other people so we need a way to distinguish who created what. To make that easy,)

[//]: # (edit `~/serverless_workshop/savealife/.chalice/config.json` to contain your first name, like so:)

[//]: # ()
[//]: # (```)

[//]: # (cat > .chalice/config.json <<EOF )

[//]: # ({)

[//]: # (  "version": "2.0",)

[//]: # (  "app_name": "$MY_FIRST_NAME-savealife",)

[//]: # (  "stages": {)

[//]: # (    "dev": {)

[//]: # (      "api_gateway_stage": "api")

[//]: # (    })

[//]: # (  })

[//]: # (})

[//]: # (EOF)

[//]: # (```)
