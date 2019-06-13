---
layout: post
title: Window functions
---

<p class="excerpt">Window functions in SQL, their syntax, benefits and coverage in MySQL and PostgreSQL.</p>

### Outline with an example

Window functions are functions applied to sets of rows. An SQL query that uses a window function processes some selected rows $S$ and for each processed row $r$ in $S$ defines a set of rows called a *window*; let's denote it $W_r$. This window provides the context for the window functions to operate within. *Windowing* means applying calculations to a set of rows and returning a single value. Let's be quick to jump to an example:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/74853d572c88c37e709d420d2409017d.js"></script></div>

The query above could yield results like the following:

```sql
id      order_date  value     rank
------- ----------- --------- ----
4       2019-03-23  56        1
2       2019-02-20  40        2
3       2019-03-23  33        3
```

In this example the selected rows $S$ are all the rows from `Sample.Table` as specified by the `SELECT` part. For each row $r$, the `OVER` clause creates its own, independent window $W_r$; here each window is the same and consists, again, of all the rows in the table, so $W_r = S$. This is because `OVER` does impose any restrictions on the windows, only ordering. Subsequently, both $r$ and $W_r$ are passed to the window function.

`RANK()` is the window function in this example. It computes the rank of a given record $r$ as one more than the number of rows in $W_r$ that have a greater ordering value than $r$.

### Syntax

The general form of a window aggregate function is `function(<args>) OVER([<partition clause>][<order clause>][<frame clause>]])`. Subsequent sections cover each part in more detail.

#### Partitioning

Specified by the `PARTITION BY` clause, partitioning restricts the scope of $W_r$ to only those rows from $S$ that have the same values in the partitioning columns as the current row $r$. 

For example, the following query would print each sales value next to its associated customer id together with its percentage share among all sales values for this customer. Here `SUM()` is the window function and `OVER` restricts $W_r$ to only those rows that have the same value in `customer_id` as $r$ does.

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/ec1836acb2658054617f094b27fa5406.js"></script></div>

#### Ordering

Ordering is defined by the `ORDER BY` clause which defines the order for the calculation within $W_r$. With ranking functions like `RANK()` or `ROW_NUMBER()` this is pretty straightforward. Compare the example in the first section; we obviously have to specify by which value we want to rank the records and this is the role of the `ORDER BY value DESC` part. 

However, for aggregate window functions like `SUM()` or `AVG()` ordering of the rows does not matter. In this case, it is only used to give meaning to the framing options which are covered in the next section.

#### Framing

Framing, like partitioning, is a filter that restricts the rows in $W_r$. It is applicable to aggregate window functions and to three of the offset functions: `FIRST_VALUE()`, `LAST_VALUE()`, and `NTH_VALUE()` and requires an ordering imposed on the window. Based on this ordering $W_r$ is framed to contain only a selection of rows. 

The window frame clause can include three parts and takes the following form `<window frame units> <window frame extent> [<window frame exclusion>]`. We shall briefly describe each part and short examples will follow in the next section. 

Frame units specify the way in which the frame is determined. It can be either `ROWS`, `RANGE` or `GROUPS`, the last one not supported by MySQL and only supported by PostgreSQL of version at least 11. The frame extent part describes the boundaries quantitatively by statements `UNBOUNDED PRECEDING/FOLLOWING`, `<n> PRECEDING/FOLLOWING` or `CURRENT ROW`. The window frame-exclusion option specifies what to do with the current row and its peers in case of ties. Has the form `EXCLUDE CURRENT ROW/GROUP/TIES/NO OTHERS` (default), not supported by MySQL, the last three supported by Postgres from version 11.

#### Framing examples

Suppose our dataset is contained within the table `Example.Data` and contains the following records:

```sql
id         value
-------    -------
1          15
2          20
3          5
4          15
5          9
```

##### Framing with ROWS

The `ROWS` option allows to indicate the bounds of the frame as an offset from $r$ in the number of rows. Let's take the example of the following query:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/ebd30730927ac29012215b7a55a33612.js"></script></div>

It would yield the following result:

```sql
 id        sum 
-------    --------
1          35
2          40
3          55
4          64
5          64
```
For each $r$ the query defines $W_r$ to be $r$ together with all preceding and one following row (in the order given by `id`). Thus for `id = 3` the `sum` is equal to `55` as a sum of `value` for `id = 1, 2, 3, 4`. One could use `EXCLUDE CURRENT ROW` to exclude `id = 3` here.

##### Framing with RANGE

The `RANGE` option defines the offsets in terms of a difference with respect to $r$'s value. Take a look at the following:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/cdf66b4435be4ab4181b29dbff661d26.js"></script></div>

```sql
id        sum 
-------   --------
3         5
5         14
1         30
4         30
2         20
```

Notice the change in ordering in both `ORDER BY`'s. Now we defined the sum to be over all the rows that have `value` at least 4 smaller than $r$ and at most equal to the value of $r$. Hence for example `id = 5` has `sum = 14` as the only rows within this range are `id = 5, 3`.

##### Framing with GROUPS

Finally, the `GROUPS` option frames the window to a number of groups of rows, each group having the same value.

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/bc1b5459528de2d660f5a6e74659dd35.js"></script>
</div>

```sql
id        sum 
-------   --------
3         14
5         39
1         50
4         50
2         20
```

Consequently, `sum` for `id = 1` is `50` as a sum of `id = 1, 4` (the same group) and `id = 2` (the following group). Had we used `EXCLUDE GROUP` here, `id = 1, 4` would be neglected in this case, while `EXCLUDE TIES` would count only one of them.

### Comparison to other approaches

These give you insights into aggregates, but you lose the details. For example, let's say we have a database of books, and each book has its weight and owner. For each book, we want to calculate the percentage of the current weight of the bookworm's total, and maybe a difference from the average. If you aggregate the data by owner, you don't have access to individual weights. A common way to handle this is to have a query that groups the data, define a table expression based on this query, and then join it with the main table to match the detail with the aggregate. Here’s a query that uses this approach:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/cdbe35ee2ac7e54c8eb14a1ae4dd4277.js"></script></div>

But what if you also need to include percentages and differences from the global aggregates (including all the owners)? You would have to create another table expression and another join. It quickly gets complicated.

Another approach is to use subqueries:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/a5c3e2000906f6bc5483a1b7477b0053.js"></script></div>

It suffers from the same problem: complication. What's more, most (if not all) SQL optimizers are not clever enough to use the fact that the subquery uses the same rows, and therefore perform multiple accesses.

Look how simple it becomes with window functions:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/7def37a0c6e3685217e5cddc8c442725.js"></script></div>

### Bibliography

1. Microsoft SQL Server 2012 High-Performance T-SQL Using Window Functions, Itzik Ben-Gan
2. <a name="wf-bib2" href="https://blog.jooq.org/2018/07/05/postgresql-11s-support-for-sql-standard-groups-and-exclude-window-function-clauses/">PostgreSQL 11's Support for SQL Standard GROUPS and EXCLUDE Window Function Clauses, blog.jooq.org</a>
3. <a name="wf-bib3" href="https://dev.mysql.com/doc/refman/8.0/en/window-functions-frames.html">Window Function Frame Specification, dev.mysql.com</a>
