# DVD Rental Business Analysis 

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning/Preparation](#data-cleaningpreparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Results](#results)
- [Recommendations](#recommendations)
- [Limitations](#limitations)
- [References](#references)

### Project Overview

This data analysis project aims to provide insights into the sales performance of a DVD rental company. By analyzing various aspects of the sales data, I seek to pinpoint specific patterns within the sales data and formulate data-driven recommendations based on these findings.

### Data Sources

Sales Data: The primary dataset used for this analysis is the "dvdrental.tar" file, containing detailed information about the business processes of the company.

### Tools

- Excel - Data Cleaning
- PG Admin 4 - Database Administration - Data Analysis
  - [Download here](https://www.pgadmin.org/download/pgadmin-4-windows/)
- PostgreSQL - Relational Database Management Systems (RDBMS) - Data Analysis
  - [Download here](https://www.postgresql.org/download/)

### Data Cleaning/Preparation

In the initial data preparation phase, I performed the following tasks:
1. Data loading and inspection
2. Handling missing values
3. Data cleaning and formatting

### Exploratory Data Analysis

EDA involved exploring the sales data to answer key questions, such as:

- Who are the staff making the highest sales?
- Who are the customers spending the most on DVD rentals?
- What patterns are showing up and what can they tell us?

### Data Analysis

```sql
SELECT SUM(p.amount), s.staff_id, s.first_name, s.last_name, s.address_id, s.store_id
FROM payment p
JOIN staff s ON s.staff_id = p.staff_id
GROUP BY s.staff_id
```
The first thing I’m noticing is that there are only two store staff. They each only work at their own separate store location. Their sales are relatively similar in total amount, although Jon has slightly higher total sales. Perhaps more information can clarify patterns? I decide to explore more.

```sql
SELECT SUM(p.amount), sta.staff_id, sta.first_name, sta.last_name, sta.address_id, sta.store_id, a.district, c.city, co.country
FROM payment p
JOIN staff sta ON sta.staff_id = p.staff_id
JOIN address a ON sta.address_id = a.address_id
JOIN city c  ON a.city_id = c.city_id
JOIN country co ON c.country_id = co.country_id
GROUP BY sta.staff_id, a.district, c.city, co.country
```
We can see that store locations are in different cities and countries. No patterns are standing out within this staff data.

Now, let’s explore the top 10 customers.

```sql
SELECT SUM(p.amount) as total_amt, c.customer_id, c.first_name, c.last_name,     c.store_id
FROM payment p
JOIN customer c ON p.customer_id = c.customer_id
GROUP BY c.customer_id
ORDER BY total_amt DESC
LIMIT 10
```
We explore information on the top customers locations. United States shows up twice, but locations are pretty spread out. Perhaps this is an online store, which would allow for the two store locations but multiple customer locations. 

```sql
SELECT co.country, ci.city, c.*, a.address, a.district, a.postal_code
FROM customer c
JOIN address a ON c.address_id = a.address_id
JOIN city ci ON a.city_id = ci.city_id
JOIN country co ON ci.country_id = co.country_id
WHERE c.customer_id IN (148, 526, 178, 137, 144, 459, 181, 410, 236, 403)
ORDER BY CASE c.customer_id
            WHEN 148 THEN 1
            WHEN 526 THEN 2
            WHEN 178 THEN 3
            WHEN 137 THEN 4
            WHEN 144 THEN 5
            WHEN 459 THEN 6
            WHEN 181 THEN 7
            WHEN 410 THEN 8
            WHEN 236 THEN 9
            WHEN 403 THEN 10
            ELSE 11
          END;
```
We explore the top customers film category preferences. Family is the top category, with foreign, horror, sci-fi, and documentary trailing in the top results.

```sql

SELECT COUNT(cat.name), cat.name
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category cat ON fc.category_id = cat.category_id
WHERE c.customer_id IN (148, 526, 178, 137, 144, 459, 181, 410, 236, 403)
GROUP BY cat.name
ORDER BY COUNT(cat.name) DESC
```

We see that there are a couple popular titles. 

```sql
SELECT COUNT(f.title), f.title
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category cat ON fc.category_id = cat.category_id
WHERE c.customer_id IN (148, 526, 178, 137, 144, 459, 181, 410, 236, 403)
GROUP BY f.title
ORDER BY COUNT(f.title) DESC
```
There are only English films in this database.

```sql
SELECT COUNT(l.name), l.name
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
JOIN language l ON f.language_id = language_id
WHERE c.customer_id IN (148, 526, 178, 137, 144, 459, 181, 410, 236, 403)
GROUP BY l.name
ORDER BY COUNT(l.name) DESC
```

DVDs rented at 4.99$ and 2.99$ are the most popular, by a difference of around 1300 and 2000 sales, respectively, ahead of 0.99$ DVDs. 

```sql
SELECT f.rental_rate, COUNT(f.rental_rate) as total_transactions
FROM customer c
JOIN payment p ON c.customer_id = p.customer_id
JOIN rental r ON p.customer_id = r.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category cat ON fc.category_id = cat.category_id
WHERE c.customer_id IN (148, 526, 178, 137, 144, 459, 181, 410, 236, 403)
GROUP BY f.rental_rate
ORDER BY total_transactions DESC
```

### Results

The analysis results are summarized as follows:
1. Jon and Mike, the only two employees, have similar total sales amounts, with Jon's total sales marginally higher, at $31,059, compared to Mike's $30,252  
2. DVDs priced at $2.99 and $4.99 had the highest volume of sales, and $0.99 had the lowest quantity of sales.
3. Your top 10 customers have purchased $150-200 worth of DVD rentals in one year.
4. Among your top 10 customers, the most popular film category is family, followed by foreign, horror, sci-fi, and documentary.

### Recommendations

Based on Data Analysis, we’d suggest you consider prioritizing adding new DVDs priced at $4.99 and $2.99 to your inventory. DVDs priced at $0.99 should be de-prioritized in considering new inventory, as they are the least popular. The existing $0.99 films should still be promoted, in order to attract customers who are drawn in by low prices. Among your top 10 customers, who’ve spent around $150-200 renting with you, family is the most popular film category. In your next promotion event, we’d recommend highlighting family movies, and foreign, horror, sci-fi, and documentaries, which were also popular choices. 

### Limitations

This is a sample database that seems to be missing some level of realism. The customers are from numerous different countries around the world, yet they only rent DVDs from two locations, in Australia and in Canada. The customers have fictional addresses. The lack of real-world data, particularly regarding staff total sales and customer locations, constrains the scope of exploratory data analysis (EDA) and diminishes the accuracy of resultant insights and trends.

### References

1. The Language of SQL 2nd Edition by Larry Rockoff
2. Data, Databases, and SQL Intermediate Course by CUNY LaGuardia



