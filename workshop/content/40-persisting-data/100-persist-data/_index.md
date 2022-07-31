+++
title = "2.1 Persisting data serverlessly"
chapter = true
weight = 100
+++

# Persisting data serverlessly

In the previous chapter we saw how to accept a JSON payload through an API Gateway and work with it using a Lambda function.
Our code so far just returns the same JSON payload. A pretty useless API I'd say. What our Lambda should be doing is 
persisting this data to a database of some sort.

## Choosing a database

Aaaaah, choices. It's always good to have a choice, but to choose well means that you should understand what are you 
choosing _for_ and what are you choosing _from_.

#### What are we choosing for - structured vs. unstructured data

Even if you never thought about it, data naturally comes in two forms: structured and unstructured. Structured data
can be best thought of as something that fits into a typical table, with rows and columns. Think phone numbers, ZIP codes,
weather data - you get the point. Easy to search through and easy to query. If anyone is thinking relational databases - 
that's it!

On the other hand, unstructured data does not (easily) fit into rows and columns. Think images, PDFs, a ZIP file. If you're
thinking NoSQL you're on the right path.

The form or the type of data you want to persist strongly dictates the choices you are able to make. Sure, you can store
a PDF file in a MySQL database but would it be easy to search through the text of that file?

#### What are we choosing from

AWS offers database solutions for both structured and unstructured data; both relational and NoSQL options are available.

Relational:
- Aurora (MySQL and PostgreSQL-compatible)
- RDS (MySQL, PostgreSQL, MSSQL)
- Redshift (based on PostgreSQL)

NoSQL:
- DynamoDB (key-value)
- ElastiCache (in-memory)
- DocumentDB (MongoDB compatible)
- and many others

![](/images/db_choices.jpg)

It is important to understand the problem you're trying to solve but also to remember the context you're
in and any (non)technical requirements/constrains you have. Page on [What are we going to build](../10-intro/100-what-is-it.html) 
lists the requirements of our system:

- Web based
- **Incurs little to no cost when unused**
- **Does not require a big team of people to maintain it**

Let's focus on those in bold text since they can affect our database choice.

## Choosing the right database

| Database                | Incurs little to no cost when unused | Does not require a big team of people to maintain it |
|-------------------------|--------------------------------------|------------------------------------------------------|
| Aurora MySQL/PostgreSQL | Yes, with caveats*                   | Yes - backups and patching are handled by AWS.       |
| RDS MySQL/PostgreSQL    | No.                                  | Yes - backups and patching are handled by AWS.       |
| Redshift                | No.                                  | Yes - backups and patching are handled by AWS.       |
| DynamoDB                | Yes.                                 | Yes.                                                 |
| ElastiCache             | No.                                  | Yes - backups and patching are handled by AWS.       |
| DocumentDB              | No.                                  | Yes - backups and patching are handled by AWS.       |

<sup>*</sup> Aurora Serverless V1 can scale to zero, but it takes up to ~30 seconds to scale back up which is not acceptable.

We will choose **DynamoDB** since it fits into our requirements perfectly: it scales to $0 when unused and is a 
completely managed solution.