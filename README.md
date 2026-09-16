# SQL-Music-Store-Analysis

# SQL Analysis: Chinook Music Store Database

Analyzing a digital music store's sales data using SQL to answer real business questions.

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
