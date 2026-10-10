# SQL Learning Journal

## Section 1: What I Learned
I already work with SQL Server, so this course was less about learning SQL from scratch and more about seeing how the same ideas behave in SQLite. I noticed the difference was aggregation. In SQLite, 

```sql
SELECT CustomerId, SUM(Total) FROM Invoice
```
runs without a GROUP BY and returns one row with CustomerId. SQL Server would throw an error. That sounds friendlier, but it can quietly give wrong answers, so I'll keep the SQL Server habit of always grouping properly. I also learned another way to concatenate strings: SQLite uses `||`, while SQL Server uses `+` or `CONCAT`.

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

## Section 3: A Mistake or Struggle

```sql
SELECT TOP 5 * FROM Customer;
```

The cause was pure SQL Server muscle memory. `TOP` is T-SQL; SQLite limits rows with `LIMIT` at the end of the query. The fix:

```sql
SELECT * FROM Customer LIMIT 5;
```

## Section 4: Connecting to the Real World

My recent task at my work, I had to mass update the registrations of a wave team (a team in a marathon). Instead of writing each UPDATE by hand, I used a regex find-and-replace to turn the list of registrations into an UPDATE statement for every one. It saved time and reduced typing mistakes.