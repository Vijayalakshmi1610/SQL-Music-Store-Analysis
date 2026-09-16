# SQL-Music-Store-Analysis

# SQL Analysis: Chinook Music Store Database

Analyzing a digital music store's sales data using SQL to answer real business questions.

## Summary
This project analyzes the Chinook digital music store database using SQL to answer 6 business questions, covering customer value, geographic revenue, genre performance, revenue trends, regional customer ranking, and churn risk. Techniques used: joins, aggregation, window functions (RANK), and CTEs.

## Tools
- SQLite (Chinook sample database)
- DB Browser for SQLite

## Business Questions & Queries

### Q1: Who are the top 10 customers by total spend?

```sql
SELECT c.FirstName, c.LastName, SUM(i.Total) as TotalSpent
FROM customers c
JOIN invoices i ON c.CustomerId = i.CustomerId
GROUP BY c.CustomerId
ORDER BY TotalSpent DESC
LIMIT 10;
```

**Result:**
| FirstName | LastName    | TotalSpent |
|-----------|-------------|------------|
| Helena    | Holý        | 49.62      |
| Richard   | Cunningham  | 47.62      |
| Luis      | Rojas       | 46.62      |
| Ladislav  | Kovács      | 45.62      |
| Hugh      | O'Reilly    | 45.62      |
| Frank     | Ralston     | 43.62      |
| Julia     | Barnett     | 43.62      |
| Fynn      | Zimmermann  | 43.62      |
| Astrid    | Gruber      | 42.62      |
| Victor    | Stevens     | 42.62      |

**Insight:** Spending among the top 10 customers is fair and evenly distributed (ranging from $42.62–$49.62), suggesting no single customer dominates revenue, the business has a healthy base of repeat high-value customers rather than reliance on one or two big spenders.

### Q2: What is total revenue by country?

```sql
SELECT BillingCountry, SUM(Total) as TotalRevenue
FROM invoices
GROUP BY BillingCountry
ORDER BY TotalRevenue DESC;
```

**Result:**
| Country | TotalRevenue |
|---|---|
| USA | 523.06 |
| Canada | 303.96 |
| France | 195.10 |
| Brazil | 190.10 |
| Germany | 156.48 |
| United Kingdom | 112.86 |

**Insight:** The USA generates significantly more revenue than any other country (72% more than Canada), and together the top 5 countries account for the large majority of total revenue, the business is heavily reliant on a small number of core markets rather than being evenly spread globally.

### Q3: Which genres generate the most revenue?

```sql
SELECT g.Name as Genre, SUM(ii.UnitPrice * ii.Quantity) as Revenue
FROM genres g
JOIN tracks t ON g.GenreId = t.GenreId
JOIN invoice_items ii ON t.TrackId = ii.TrackId
GROUP BY g.Name
ORDER BY Revenue DESC;
```

**Result:**
| Genre | Revenue |
|---|---|
| Rock | 826.65 |
| Latin | 382.14 |
| Metal | 261.36 |
| Alternative & Punk | 241.56 |
| TV Shows | 93.53 |
| Jazz | 79.20 |

**Insight:** Rock dominates revenue by a wide margin, more than double the next closest genre (Latin) and over 3x Metal. Rock, Latin, and Metal alone account for the majority of total genre revenue, suggesting inventory/marketing focus should stay concentrated on these top genres rather than spreading evenly across all 24.

### Q4: What is the monthly revenue trend?

```sql
SELECT strftime('%Y-%m', InvoiceDate) as Month, SUM(Total) as Revenue
FROM invoices
GROUP BY Month
ORDER BY Month;
```

**Result:**
| Month | Revenue |
|---|---|
| 2009-01 | 35.64 |
| 2009-02 | 37.62 |
| 2009-03 | 37.62 |
| ... | ... |
| 2013-11 | 49.62 |
| 2013-12 | 38.62 |

**Insight:** Revenue is remarkably flat month-to-month ($37.62) across all 5 years, with only occasional small spikes (eg, Jan 2010: $52.62, Apr 2011: $51.62). This is unusual for a real retail business — steady, near-identical monthly revenue like this suggests the dataset is synthetic/sample data rather than real transactional data, an important observation to flag when presenting findings, since real businesses typically show seasonality, growth, or decline.

### Q5: Rank customers by spend within each country (window function)

```sql
SELECT 
    c.FirstName, 
    c.LastName, 
    c.Country,
    SUM(i.Total) as TotalSpent,
    RANK() OVER (PARTITION BY c.Country ORDER BY SUM(i.Total) DESC) as RankInCountry
FROM customers c
JOIN invoices i ON c.CustomerId = i.CustomerId
GROUP BY c.CustomerId
ORDER BY c.Country, RankInCountry;
```

**Result (sample):**
| FirstName | LastName | Country | TotalSpent | RankInCountry |
|---|---|---|---|---|
| Richard | Cunningham | USA | 47.62 | 1 |
| Frank | Ralston | USA | 43.62 | 2 |
| Julia | Barnett | USA | 43.62 | 2 |
| Helena | Holý | Czech Republic | 49.62 | 1 |
| Luís | Gonçalves | Brazil | 39.62 | 1 |
| ... | ... | ... | ... | ... |

**Insight:** Using `RANK()` partitioned by country identifies the top customer in each market rather than just globally — useful for targeted retention campaigns (eg, VIP offers per region). Several countries (Argentina, Australia, Belgium, etc.) have only one customer, so "rank 1" there isn't meaningful; this is a useful caveat to note when presenting findings, since applying the same ranking logic to markets with very different customer counts can be misleading if not called out.

### Q6: Which customers haven't purchased in the last 6 months? (churn risk — CTE)

```sql
WITH LastPurchase AS (
    SELECT CustomerId, MAX(InvoiceDate) as LastPurchaseDate
    FROM invoices
    GROUP BY CustomerId
)
SELECT c.FirstName, c.LastName, lp.LastPurchaseDate
FROM customers c
JOIN LastPurchase lp ON c.CustomerId = lp.CustomerId
WHERE lp.LastPurchaseDate < date((SELECT MAX(InvoiceDate) FROM invoices), '-6 months')
ORDER BY lp.LastPurchaseDate;
```

**Result (sample):**
| FirstName | LastName | LastPurchaseDate |
|---|---|---|
| Puja | Srivastava | 2012-05-30 |
| Niklas | Schröder | 2012-06-30 |
| Leonie | Köhler | 2012-07-13 |
| ... | ... | ... |
| Astrid | Gruber | 2013-06-19 |

**Insight:** 28 customers (roughly half the customer base) haven't purchased in 6+ months relative to the dataset's last invoice date. This CTE approach, first finding each customer's last purchase date, then filtering against a rolling cutoff is a reusable pattern for churn/retention analysis and could be automated into a recurring report to flag at-risk customers for a win-back campaign.

## Key Takeaways
- Revenue is concentrated in the USA and a handful of top countries
- Rock, Latin, and Metal genres drive the majority of revenue
- Customer spend is fairly evenly distributed at the top, rather than dominated by one whale customer
- 50% of customers show signs of churn risk (no purchase in 6+ months) a strong candidate for a retention campaign
