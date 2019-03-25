---
layout: post
title: Window functions
---

<p class="excerpt">Window functions in MySQL and PostgreSQL.</p>

### What is it?

Window functions are functions applied to sets of rows defined by an OVER() clause. This set of rows is called a *window* and it provides context for the functions to operate in.

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/74853d572c88c37e709d420d2409017d.js"></script></div>

The above query would yield results like:

```sql
id      date        value     rank
------- ----------- --------- ----
4       2019-03-23  56        1
2       2019-02-20  40        2
3       2019-03-23  33        3
```
The OVER clause defines a window for the function *with respect to* each row in the result set of the query. In this case, the rank of a given row is computed as one more than the number of rows in the relevant set that have a greater ordering value than the current row, and the relevant set is the set of all the entries (as the OVER clause does not restrict it in any way).

### Why?

#### Why not use a simple grouped query?

These give you insights into aggregates, but you lose the details. For example, let's say we have a database of books, and each book has its weight. For each book, we want to calculate the percentage of the current weight of the bookworm total, and maybe a difference from the average. If you aggregate the data by owner, you don't have access to individual weights. A common way to handle this is to have a query that groups the data, define a table expression based on this query, and then join it with the main table to match the detail with the aggregate. Here’s a query that uses this approach:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/cdbe35ee2ac7e54c8eb14a1ae4dd4277.js"></script></div>

But what if you also need to include percentages and differences from the global aggregates (including all the owners)? You would have to create another table expression and another join. It quickly gets complicated.

Another approach is to use subqueries:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/a5c3e2000906f6bc5483a1b7477b0053.js"></script></div>

It suffers from the same problem: complication. What's more, most (if not all) SQL optimizers are not clever enough to use the fact that the subquery uses the same rows, and therefore perform multiple accesses.

Look how simple it becomes with window functions:

<div class="gist-wrapper"><script src="https://gist.github.com/slamii/7def37a0c6e3685217e5cddc8c442725.js"></script></div>
