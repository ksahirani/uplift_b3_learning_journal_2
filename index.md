# SQL Learning Journal #2

*Reflecting on what I learned using SQL for the Chinook database*

---

## Section 1: What I Learned

In this module I covered SELECT and filtering, sorting, aggregation with GROUP BY, joins, subqueries, and CTEs, all on the Chinook database. Three ideas finally made sense to me when I explained them in my own words.

**WHERE vs HAVING.** WHERE filters individual rows before any grouping happens. HAVING filters groups after GROUP BY and aggregation, which is why it can use things like `AVG()` or `COUNT()`, and WHERE cannot.

**Non-correlated vs correlated subqueries.** A non-correlated subquery runs once on its own, like `WHERE unit_price > (SELECT AVG(unit_price) FROM track)`. A correlated subquery refers to the outer query, so it re-runs for each outer row. For example, comparing each track to the average of its own album.

**CTEs.** A CTE, written with `WITH`, gives a subquery a name so I can use it like a temporary table. It makes long queries easier to read and reuse than deeply nested subqueries.

---

## Section 2: A Query I Am Proud Of

```sql
SELECT al.title AS album_title,
       AVG(t.milliseconds) AS avg_length_ms
FROM track t
JOIN album al ON t.album_id = al.album_id
GROUP BY al.album_id, al.title
ORDER BY avg_length_ms DESC;
```

The database processes this in a logical order, not the order I wrote it. First, `FROM track t` picks the table. Then `JOIN album al ...` matches each track to its album. `GROUP BY` collects the rows into one group per album. `SELECT` then calculates the average track length for each group. Finally, `ORDER BY` sorts the albums from longest average track to shortest.

I'm proud of it because it combines a join, grouping, an aggregate, and sorting in one query, and I grouped by `album_id` as well as `title` so two albums with the same title are not merged. The business question it answers is which albums have the longest tracks on average. A manager could use that to plan playlists or decide which albums suit long-form listening.

---

## Section 3: A Mistake or Struggle

My biggest struggle was a query that worked in SQLite but failed in PostgreSQL:

```sql
SELECT CustomerId, FirstName, LastName FROM Customers;
```

*(Table customer not found)*

The cause was identifier case sensitivity. SQLite ignores letter case for table and column names. PostgreSQL folds unquoted names to lowercase, but the Chinook version I imported had created the tables with quoted mixed-case names like `"Customer"`. My unquoted query was therefore looking for `customer`, which didn't exist.

I found it by comparing how the tables were named in the DBeaver navigator with what I typed. I fixed it by either quoting the exact name (`"Customer"`) or using the lowercase snake_case version of the schema, which I switched to. Next time I'll check the table and column names in the database before writing queries, and I'll make sure my editor is connected to the right database.

---

## Section 4: Connecting to the Real World

Take an online music store, like the one Chinook models. It could answer two questions with SQL:

1. **Who are our top 10 customers by total spending?** This joins `customer`, `invoice`, and `invoice_line`, then uses `SUM()` with `GROUP BY` and `ORDER BY ... LIMIT 10`.
2. **Which tracks cost more than the average for their genre?** This needs a correlated subquery comparing each track's price to the average price of its own genre.

SQL is still widely used because most business data lives in relational databases, the language is standardised across systems, and it's built for filtering, joining, and summarising large amounts of data quickly. It also connects to my machine-learning work. I can use SQL to build features and load data into pandas with `pd.read_sql()`.

---

## Section 5: Self-Assessment

*(Suggested ratings. Change them to match your own honest view.)*

| Topic | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Basic SELECT, WHERE, ORDER BY | | | | X | |
| Aggregation (COUNT, SUM, AVG, GROUP BY, HAVING) | | | | X | |
| Joins (2 tables) | | | | X | |
| Joins (3 or more tables) | | | X | | |
| Subqueries | | | X | | |
| CTEs (WITH) and window functions | | X | | | |
| CREATE, INSERT, UPDATE, DELETE | | | X | | |

My two highest ratings are basic SELECT and aggregation. I can write these without notes, and I used `GROUP BY` with a join to get the average track length per album. My two lowest are CTEs with window functions, and subqueries. I needed help writing a CTE for the Chinook exercise, and I first mixed up a per-album average with a global average, which showed I still confuse correlated and non-correlated subqueries. Window functions are the area I've practised least.

---

## Section 6: Goals and Next Steps

I want to strengthen subqueries and CTEs. I'll practise by taking the Chinook queries I wrote with nested subqueries and rewriting each as a CTE, then checking that the results match.

A topic I'm curious about is **window functions**, because they can rank or compare rows without collapsing them into groups, and they look useful for feature engineering in ML.

My measurable goal: by next week, I will rewrite three of my subquery queries as CTEs, write one correlated subquery on my own, and explain the difference between them in a short note.
