---
sourcePath: en/ydb/ydb-docs-core/en/core/faq/_includes/yql.md
---
# YQL

### General questions {#common}

#### How do I select table rows by a list of keys? {#explicit-keys}

You can select table rows based on a specified list of table primary key (or key prefix) values using the `IN` operator:

```sql
DECLARE $keys AS List<UInt64>;

SELECT * FROM some_table
WHERE Key1 IN $keys;
```

If a selection is made using a composite key, the query parameter must have the type of a list of tuples:

```sql
DECLARE $keys AS List<Tuple<UInt64, String>>;

SELECT * FROM some_table
WHERE (Key1, Key2) IN $keys;
```

To select rows effectively, make sure that the value types in the parameters match the key column types in the table.

#### Is search by index performed for conditions containing the LIKE operator? {#like-index}

You can only use the `LIKE` operator to search a table index if it specifies a row prefix:

```sql
SELECT * FROM string_key_table
WHERE Key LIKE "some_prefix%";
```

#### Why does a query return only 1000 rows? {#result-rows-limit}

1000 rows is the response size limit per YQL query. If a response is shortened, it is flagged as `Truncated`. To output more table rows, you can use [paginated output](../../best_practices/paging.md) or the `ReadTable` operation.

#### What do I do if I get the Datashard: Reply size limit exceeded error in response to a query? {#reply-size-exceeded}

This error means that, as a query was running, one of the participating data shards attempted to return over 50 MB of data, which exceeds the allowed limit.

Recommendations:

* A general recommendation is to reduce the amount of data processed in a transaction.
* If a query involves a `Join`, it is a good idea to make sure that the method used is [Index lookup Join](#index-lookup-join).
* If a simple selection is performed, make sure that it is done by keys, or add `LIMIT` in the query.

#### What do I do is I get the "Datashard program size limit exceeded" in response to a query? {#program-size-exceeded}

This error means that the size of a program (including parameter values) exceeded the 50-MB limit for one of the data shards. In most cases, this indicates an attempt to write over 50 MB of data to database tables in a single transaction. All modifying operations in a transaction such as `UPSERT`, `REPLACE`, `INSERT`, or `UPDATE` count as records.

You need to reduce the total size of records in one transaction. Normally, we don't recommend combining queries that logically don't require transactionality in a single transaction. When adding/updating data in batches, we recommend reducing the size of one batch to values not exceeding a few megabytes.

#### How do I update only those values whose keys are not in the table? {#update-non-existent}

You can use the `LEFT JOIN` operator to identify the keys a table is missing and update their values:

```sql
DECLARE $values AS List<Struct<Key: UInt64, Value: String>>;

UPSERT INTO kv_table
SELECT v.Key AS Key, v.Value AS Value
FROM AS_TABLE($values) AS v
LEFT JOIN kv_table AS t
ON v.Key = t.Key
WHERE t.Key IS NULL;
```

### Join operations {#joins}

#### Are there any specific features of Join operations? {#join-operations}

A `Join` in {{ ydb-short-name }} is performed using one of the two methods below:

* Common Join.
* Index Lookup Join.

#### Common Join {#common-join}

The contents of both tables (to the left and to the right of `Join`) are sent to the requesting node which applies the operation to the totality of the data. This is the most generic way of performing a `Join` that is used whenever other optimizations are unavailable. For large tables, this method is either slow or doesn't work in general due to exceeding the data transfer limits.

#### Index Lookup Join {#index-lookup-join}

For rows on the left of `Join`, relevant values are looked up to the right. You use this method whenever the right part is a table and the `Join` key is its primary or secondary index key prefix. In this method, limited selections are made from the right table instead of full reads. This lets you use it when working with large tables.

{% note info %}

For most OLTP queries, we recommend using Index Lookup Join with a small size of the left part. These operations read little data and can be performed efficiently.

{% endnote %}

#### How do I Join data from query parameters? {#constant-table-join}

You can use query parameter data as a constant table. To do this, use the `AS_TABLE` modifier with a parameter whose type is a list of structures:

```sql
DECLARE $data AS List<Struct<Key1: UInt64, Key2: String>>;

SELECT * FROM AS_TABLE($data) AS d
INNER JOIN some_table AS t
ON t.Key1 = d.Key1 AND t.Key2 = d.Key2;
```

There is no explicit limit on the number of entries in the constant table, but mind the standard limit on the total size of query parameters (50 MB).

#### What is the best way to implement a query like (key1, key2) IN ((v1, v2), (v3, v4), ...)? {#key-pairs-in}

It's better to write it using a JOIN with a constant table:

```sql
$keys = AsList(
    AsStruct(1 AS Key1, "One" AS Key2),
    AsStruct(2 AS Key1, "Three" AS Key2),
    AsStruct(4 AS Key1, "One" AS Key2)
);

SELECT t.* FROM AS_TABLE($keys) AS k
INNER JOIN table1 AS t
ON t.Key1 = k.Key1 AND t.Key2 = k.Key2;
```

### Transactions {#transactions}

#### How efficient is it to run multiple queries in a transaction? {#transaction-queries}

When multiple queries are run sequentially, the total transaction latency may be greater than when the same operations are executed within a single query. This is primarily due to additional network latency for each query. Therefore, if a transaction doesn't need to be interactive, we recommend formulating all operations in a single YQL query.

#### Is a separate query atomic? {#atomic-query}

In general, YQL queries can be executed in multiple consecutive phases. For example, a Join query can be executed in two phases: reading data from the left and right table, respectively. This aspect is important when you run a query in a transaction with a low isolation level (`online_read_only`), as in this case, data between execution phases can be updated by other transactions.

