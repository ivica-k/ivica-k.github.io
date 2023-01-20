---
title: "6.1 Get all donors from a city"
chapter: true
weight: 10
---

# Get all donors from a city
As a short reminder, data structures that we're saving into DynamoDB are:

##### Donor

```json
{
  "first_name": "itsme",
  "email": "itsme@server.com",
  "blood_type": "A-",
  "city": "Amsterdam"
}
```

##### Blood donation event

```json
{
  "city": "Amsterdam",
  "datetime": "2022-04-06T12:00:00",
  "address": "Main street"
}
```

Our job is to perform a query on the table that will return to us all the donors from a single city. Roughly translated
into SQL:

```sql
SELECT * FROM donors
WHERE `city` = 'Amsterdam';
```

A typical one-to-many relationship.

DynamoDB does support filtering a result set from the `Scan` operation, but the filtering is applied after a `Scan`
finishes. This results in a `Scan` operation that may potentially consume many read capacity units (and cost quite a bit
in the long run). On top of that, the maximum amount of data a single `Scan` operation can return is 1MB. If your table
has more than 1MB of data to return, the result set can be paginated.

With all that being said, I would prefer if we again used the `Query` operation since it offers surgical precision and
reads and returns just the right data. To be able to offer this precision it also requires precision from us. To `Query`,
we must provide the partition key (`PK`) value. In case of donors, the `PK` has a format of `DONOR#abcd1234`. But if we were
to provide that `PK` value to the query operation, the result would be a single donor that matches. That is not what we
want.

## Get all donors from a city

Our goal is to fetch all donors for a specific city. Looking at the data structures for a donor and a donation, they have
the `city` field in common. As already established, this is a one-to-many relationship - one city has many donors. Modeling
a one-to-many relationship with DynamoDB can be done in [multiple ways](https://www.alexdebrie.com/posts/dynamodb-one-to-many/).

Secondary indexes are a way to provide additional query patterns over the same data set. A _secondary index_
is a data structure that contains a subset of attributes from a table, along with an alternate key to support Query 
operations. You can retrieve data from the index using a `Query`, in much the same way as you use `Query` with a table. 
A table can have multiple secondary indexes, which give your applications access to different query patterns.

The following page will go through the process of creating a secondary index and showing how it can be used for an 
additional query pattern.