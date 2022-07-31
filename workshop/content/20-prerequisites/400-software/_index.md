+++
title = "Other software"
chapter = true
weight = 400
+++

## Code editor

You may use whichever code editor suits you, we don't discriminate:
- VSCode
- PyCharm
- `vim`
- `kate`

Jump to the [Donor sign-up page](../30-donor-signup.html) **if you ran the `./workshop_setup.sh` script** from the
[Project setup](./100-setup.html).
***

## HTTP client

You may use whichever HTTP client suits you, as long as you are comfortable making `GET` and `POST` requests with it.

Some popular examples are:
 - [Postman](https://www.postman.com/)
 - `curl`
- [httpie](https://httpie.io/) - examples in the workshop will use it

You can add `httpie` to `requirements-dev.txt`. To make use of environment variables needed for development we will be 
using the [python-dotenv](https://pypi.org/project/python-dotenv/) project.

```bash
cat >> requirements-dev.txt <<EOF
httpie==3.1.0
python-dotenv==0.20.0
EOF
```

Install all the development requirements before we dive into coding

```bash
pip install -r requirements-dev.txt
```

## Operating system

Again, whatever works for you, as long as you are able to:
- set environment variables
- start CLI applications

All code snippets and commands were tested on Linux and MacOS with their native terminal applications and shells. On Windows,
[GitBash](https://git-scm.com/downloads) was used as it mimics a `*NIX` style terminal application very well.
[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) was also reported to work just fine.