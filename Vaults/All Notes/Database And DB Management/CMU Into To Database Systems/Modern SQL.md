_____
**Created**: 28-10-2024 03:57 pm
**Status**: In Progress
**Tags**: #SQL #DatabaseSystem [[SQL]]
**References**: [Introduction to Database Systems](https://youtube.com/playlist?list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&si=t4vHmPBTCxAqTUjT)
______

SQL is based on **bags(duplicates)** and not **sets(no duplicates).** Bag are unordered and can have duplicates, while sets on the other hand are unordered but cannot have duplicates.

We've got DML: Data Manipulation Language, DDL: Data Definition Language, and DCL: Data Control Language.
Additionally there are things like View Definitions, Integrity and Referential Constraints, and [[Transactions]].

### Aggregates
These are functions that return a single value.
The `HAVING` clause, lets you filer out data after aggregation.

### Window Functions
```sql
-- Row numbers are incremental in this format.
SELECT *, ROW_NUMBER OVER() AS row_num FROM some_table

-- PARTITION BY is to specify gorup
-- Row numbers now reset between gorups/partitions
SELECT cid, sid, ROW_NUMBER() OVER(PARTITION BY cid) FROM enrolled ORDER BY cid

-- FIND the student with the second highest grade for each course
SELECT * FROM(
	SELECT *, RANK() OVER (PARTITION BY cid ORDER BY grade ASC) AS rank
	FROM enrolled
) AS ranking
WHERE ranking.rank = 2
```

### Nested Queries
As the title hints it involves, invoking a query inside of another query to generally get parts of more complicated queries and compose them together.
The nested queries can appear *almost anywhere inside a query*.

Lets see a dumb way to use nested queries:
```sql
SELECT sid, (SELECT name FROM student AS s WHERE s.id = e.sid) AS name
FROM enrolled AS e;
```
It seems like the correct thing until your production burns down because of the overuse of the DB resources. **There is a world where the DB will run the nested `SELECT` statement for every tuple inside the `enrolled` table. Now that is a recipe for disaster.** The database does run optimisations and more often than not it will become a join, but it does not mean that it always will, better be explicit. Better to just use a `JOIN`.

### Lateral JOINS
The `LATERAL` operator allows a nested query to reference attributes in other nested queries that precede it.
It can be thought of as a `for loop` that allows you to invoke another query for each tuple in a table.
```sql
SELECT * FROM (SELECT 1 AS x) AS t1, LATERAL (SELECT t1.x + 1 AS y) AS t2;
```

Lets see an example of it in action that is actually useful.
We need to calculate the number of students enrolled in each course and the average GPA. Sort them by enrolment count in descending order.
```sql
SELECT * FROM course AS c
	LATERAL (SELECT COUNT(*) AS cnt FROM enrolled WHERE enrolled.cid = c.cid) AS q1
	LATERAL (SELECT AVG(GPA) AS avg FROM student AS s JOIN enrolled AS e on s.id = e.id WHERE e.cid = c.cid) as q2
```