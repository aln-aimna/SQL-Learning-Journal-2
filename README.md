# SQL Learning Journal

## Section 1: What I Learned

I already work with SQL Server, so this course was less about learning SQL from scratch and more about seeing how the same ideas behave in SQLite. The biggest difference was aggregation. In SQLite, `SELECT CustomerId, SUM(Total) FROM Invoice` runs without a GROUP BY and returns one row with an arbitrary CustomerId. SQL Server would throw an error. That sounds friendlier, but it can quietly give wrong answers, so I'll keep the SQL Server habit of always grouping properly. I also learned another way to concatenate strings: SQLite uses `||`, while SQL Server uses `+` or `CONCAT`.

The concept that finally clicked is that CTEs and subqueries aren't competing options. A CTE is a named result I build first and can reuse, and a subquery returns a value or list inside another query. They work well together. I also remembered temp tables like `#table1` in SQL Server. A CTE lives for one statement, while a temp table lasts for the session, so it's better when I need the same result across several queries.

## Section 2: A Query I Am Proud Of

Chinook is a great database for learning SQL, and creating it in SQLite is lightweight with no server setup.

```sql
WITH CustomerSpend AS (
    SELECT c.CustomerId,
           c.FirstName || ' ' || c.LastName AS CustomerName,
           c.Country,
           SUM(i.Total) AS TotalSpent
    FROM Customer c
    JOIN Invoice i ON c.CustomerId = i.CustomerId
    GROUP BY c.CustomerId, c.FirstName, c.LastName, c.Country
)
SELECT CustomerName, Country, ROUND(TotalSpent, 2) AS TotalSpent
FROM CustomerSpend
WHERE TotalSpent > (SELECT AVG(TotalSpent) FROM CustomerSpend)
ORDER BY TotalSpent DESC;
```

The CTE `CustomerSpend` joins Customer and Invoice, groups by customer, and sums purchases, giving one row per customer. The main query then filters with WHERE, using a subquery that calculates the average customer spend as a single value. Logically, the database builds the CTE first (FROM/JOIN, GROUP BY, SELECT), then runs the outer FROM, WHERE, SELECT, and ORDER BY.

The business question is: *which customers spend more than the average customer?* A manager could use this for loyalty offers or to see which countries have the most high-value customers.

[Add one result row, e.g. top customer and amount.]

## Section 3: A Mistake or Struggle

```sql
SELECT TOP 5 * FROM Customer;
```

Result: [paste your exact error, likely `near "5": syntax error`].

The cause was pure SQL Server muscle memory. `TOP` is T-SQL; SQLite limits rows with `LIMIT` at the end of the query. The fix:

```sql
SELECT * FROM Customer LIMIT 5;
```

It was a small error, but it showed me that knowing one dialect well can work against me in another. The same thing happened with GROUP BY: SQLite accepted a query that SQL Server would reject. Next time, before running anything, I'll ask myself which database I'm on, and I'll keep a short cheat sheet of differences: `TOP` vs `LIMIT`, `+` vs `||`, `#temp` tables vs `CREATE TEMP TABLE`. I also learned to read the error message first instead of retyping the query and hoping.

## Section 4: Connecting to the Real World

At work, I had to mass update the registrations of a wave team (a team in a marathon). Instead of writing each UPDATE by hand, I used a regex find-and-replace to turn the list of registrations into an UPDATE statement for every one. It saved time and reduced typing mistakes.

The same data could answer two business questions:

1. How many registrations does each team have? (join Registration to Team, then GROUP BY and COUNT)
2. Which teams have more registrations than the average team? (CTE plus subquery, like my Chinook query)

SQL is still widely used because it's standardized, readable, and reliable for structured data, and the skills transfer across SQL Server, SQLite, and other databases.