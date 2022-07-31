+++
title = "2.2 DynamoDB 101"
chapter = true
weight = 200
+++

# DynamoDB 101

Some contents of this page was copied from [Alex DeBries workshop on DynamoDB](https://github.com/alexdebrie/dynamodb-workshop).
Alex is the authority on DynamoDB and his [DynamoDB book](https://www.dynamodbbook.com/) is a must-read for anyone designing
data models on DynamoDB.

## What is DynamoDB

By its definition, DynamoDB is a fully-managed NoSQL database. To quote the DynamoDB book:

> DynamoDB is a NoSQL database. But NoSQL is a terrible name, as
it only describes what a database isn’t (it’s not a database that uses
the popular SQL query language used by traditional relational
databases), rather than what it is. And NoSQL databases come in a
variety of flavors. There are document databases like MongoDB,
column stores like Cassandra, and graph databases like Neo4J or
Amazon Neptune.

You can think of DynamoDB as a super-charged key-value store: like a bookshelf filled with phonebooks. Each phonebook is a 
binary tree that can be searched with great speeds, and finding a phonebook to search through is quick and deterministic
because the bookshelf is a key-value store (like a Python dictionary) so lookups are quick since they're based on a hash.

## Terminology

**Table**: A table is a grouping of records that conceptually belong together. It may contain multiple entity types:
Donors and blood donation Events may be stored in a single table (and they will be).

**Item**: An item is a single record in a table. You may think of it as a row in a SQL database.

**Attributes**: Every item in a DynamoDB table consists of attributes. They are typed data values. If you have and item
representing a Donor, it will probably have an attribute named `first_name` of type `string` with a value of `Ivica`.

## Keys

DynamoDB is a schemaless database. Unlike relational databases, this means you don't need to 
specify the names and types of all attribute that your items will have. Instead, you will manage your schema within your
application code. Being schemaless allows for greater flexibility when dealing with sparse data or when the data model 
you're persisting changes over time.

However, you do need to define a primary key for your table. Every item in your table must have the primary key for the 
table, and each item in your table is uniquely identifiable by the primary key.

There are two types of primary keys:
- Simple: A primary key that consists of a single element (the partition key);
- Composite: A primary key that consists of two elements (a partition key and a sort key).

## Throughput

With traditional databases, you often spin up servers. You might specify CPU, RAM, and networking settings for your 
instance. You need to estimate your traffic and make guesses as to how that translates to computing resources.

With DynamoDB, it's different. You pay for throughput directly rather than computing resources. 
This is split into:

- Read Capacity Units (RCUs), which refers to a strongly-consistent read of 4KB of data;
- Write Capacity Units (WCUs), which refers to a write of 1KB of data.

This means than when choosing a database, or a table to be more precise, you don't choose the amount of compute resources
such as CPU and RAM - you choose how many read and write operations you want to do per second.

There are two throughput modes you can use with DynamoDB:
- Provisioned throughput: You specify the number of RCUs and WCUs you want available on a per-second basis;
- On-demand: You are charged on a per-request basis for each read and write you make. You don't need to specify the amount
you want ahead of time.

On a fully-utilized basis, on-demand billing is more expensive than provisioned throughput. However, it's difficult to 
get full utilization or anything close to it, particularly if your traffic patterns vary over the time of day or day of 
week. Many people actually save money with on-demand, while also reducing the amount of capacity planning and 
adjustments they need to do.

## Billing

To better understand how billing works we first need to understand the two capacity modes:
- on-demand mode;
- provisioned mode.

#### On-demand mode

With on-demand capacity mode, DynamoDB charges you for the data reads and writes your application performs on your 
tables. You do not need to specify how much read and write throughput you expect your application to perform because 
DynamoDB instantly accommodates your workloads as they ramp up or down.

On-demand capacity mode might be best if you:

- Create new tables with unknown workloads;
- Have unpredictable application traffic;
- Prefer the ease of paying for only what you use.

#### Provisioned mode

With provisioned capacity mode, you specify the number of reads and writes per second that you expect your application 
to require. You can use auto-scaling to automatically adjust your table’s capacity based on the specified utilization 
rate to ensure application performance while reducing costs.

Provisioned capacity mode might be best if you:

- Have predictable application traffic;
- Run applications whose traffic is consistent or ramps gradually;
- Can forecast capacity requirements to control costs.

For this application we will be using the **on-demand** capacity mode.

## Indexes

What if you need to allow multiple, different access pattern for a certain type of items? How can you enable these 
patterns with only a single primary key? That is where secondary indexes come into the picture.

> There are two types of secondary indexes -- global and local. In almost all occasions, you'll want to use a global 
> secondary index. For the rest of this lesson, We'll use "global secondary index" and "secondary index" interchangeably.

A secondary index is something you create on your DynamoDB table that gives you additional access patterns on the items 
in your table. When you add a secondary index to your table, you will declare the primary key schema for the secondary 
index. When an item is written into your table, DynamoDB will check if the item has the attributes for your secondary 
index's primary key schema. If it does, the item will be copied into the secondary index with the primary key for the 
secondary index. You can then issue read requests against your secondary index to access items with secondary 
access patterns.

In essence, a secondary index gives you an additional, read-only view on your data.