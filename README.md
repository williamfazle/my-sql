# SQL Interview Questions

> 50 SQL questions covering beginner, intermediate, and advanced topics, with solutions using a sample ecommerce dataset.

---

## Section 1 — Basics (10 questions)
1. Retrieve all columns from the `users` table.
```sql
SELECT * FROM users;
```
2. List all products with a price greater than 1000.
```sql
SELECT * FROM products WHERE price > 1000;
```
3. Count the total number of orders in the last month.
```sql
SELECT COUNT(*) FROM orders WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH);
```
4. List distinct product categories.
```sql
SELECT DISTINCT category FROM products;
```
5. Find users who signed up after '2025-01-01'.
```sql
SELECT * FROM users WHERE signup_date > '2025-01-01';
```
6. Show the first 5 orders sorted by `order_date`.
```sql
SELECT * FROM orders ORDER BY order_date ASC LIMIT 5;
```
7. Find the minimum and maximum price from the `products` table.
```sql
SELECT MIN(price) AS min_price, MAX(price) AS max_price FROM products;
```
8. Show all orders where `quantity` > 2.
```sql
SELECT * FROM orders WHERE quantity > 2;
```
9. Replace NULL email values in users with 'noemail@example.com'.
```sql
SELECT id, name, COALESCE(email, 'noemail@example.com') AS email, signup_date FROM users;
```
10. Concatenate first and last name columns to form a full name.
```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
```

## Section 2 — Aggregation & Grouping (10 questions)
11. Count the number of orders per user.
```sql
SELECT user_id, COUNT(*) AS order_count FROM orders GROUP BY user_id;
```
12. Calculate total revenue per product.
```sql
SELECT product_id, SUM(quantity * price) AS revenue
FROM orders o JOIN products p ON o.product_id = p.id
GROUP BY product_id;
```
13. Find the average order value.
```sql
SELECT AVG(total) FROM (
  SELECT SUM(quantity * price) AS total
  FROM orders o JOIN products p ON o.product_id = p.id
  GROUP BY o.id
) t;
```
14. List the top 3 products by total quantity sold.
```sql
SELECT product_id, SUM(quantity) AS total_quantity FROM orders GROUP BY product_id ORDER BY total_quantity DESC LIMIT 3;
```
15. Count users per signup month.
```sql
SELECT DATE_FORMAT(signup_date, '%Y-%m') AS month, COUNT(*) AS users_count FROM users GROUP BY month;
```
16. Find the total revenue per category.
```sql
SELECT p.category, SUM(o.quantity * p.price) AS revenue
FROM orders o JOIN products p ON o.product_id = p.id
GROUP BY p.category;
```
17. List categories where total sales > 10000.
```sql
SELECT p.category, SUM(o.quantity * p.price) AS revenue
FROM orders o JOIN products p ON o.product_id = p.id
GROUP BY p.category
HAVING revenue > 10000;
```
18. Find the number of products with price between 500 and 5000.
```sql
SELECT COUNT(*) FROM products WHERE price BETWEEN 500 AND 5000;
```
19. Show the total revenue and total quantity sold per day.
```sql
SELECT order_date, SUM(quantity * p.price) AS revenue, SUM(quantity) AS total_qty
FROM orders o JOIN products p ON o.product_id = p.id
GROUP BY order_date;
```
20. Calculate the percentage contribution of each product category to total revenue.
```sql
SELECT category, SUM(o.quantity*p.price)/SUM(SUM(o.quantity*p.price)) OVER() *100 AS pct_contribution
FROM orders o JOIN products p ON o.product_id=p.id
GROUP BY category;
```

## Section 3 — Joins (10 questions)
21. List all users along with their total spend.
```sql
SELECT u.id, u.name, SUM(o.quantity*p.price) AS total_spent
FROM users u LEFT JOIN orders o ON u.id=o.user_id
LEFT JOIN products p ON o.product_id=p.id
GROUP BY u.id;
```
22. Show all orders along with product names and categories.
```sql
SELECT o.id, u.name AS user, p.name AS product, p.category, o.quantity, o.order_date
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN products p ON o.product_id = p.id;
```
23. Find users who have never placed an order.
```sql
SELECT u.id, u.name FROM users u
LEFT JOIN orders o ON u.id=o.user_id
WHERE o.id IS NULL;
```
24. List orders and user names where quantity > 1.
```sql
SELECT o.id, u.name, o.quantity FROM orders o JOIN users u ON o.user_id=u.id WHERE o.quantity > 1;
```
25. Show products that have never been ordered.
```sql
SELECT p.id, p.name FROM products p
LEFT JOIN orders o ON p.id=o.product_id
WHERE o.id IS NULL;
```
26. List top 5 users by number of orders.
```sql
SELECT u.id, u.name, COUNT(o.id) AS orders_count
FROM users u JOIN orders o ON u.id=o.user_id
GROUP BY u.id
ORDER BY orders_count DESC LIMIT 5;
```
27. Calculate average quantity per product across all orders.
```sql
SELECT product_id, AVG(quantity) AS avg_quantity FROM orders GROUP BY product_id;
```
28. Find users who purchased products from more than one category.
```sql
SELECT u.id, u.name
FROM users u
JOIN orders o ON u.id=o.user_id
JOIN products p ON o.product_id=p.id
GROUP BY u.id
HAVING COUNT(DISTINCT p.category) > 1;
```
29. List each user and the most expensive product they purchased.
```sql
SELECT u.id, u.name, MAX(p.price) AS max_price
FROM users u
JOIN orders o ON u.id=o.user_id
JOIN products p ON o.product_id=p.id
GROUP BY u.id;
```
30. Find users who bought the same product more than once.
```sql
SELECT user_id, product_id, COUNT(*) AS times_bought FROM orders GROUP BY user_id, product_id HAVING times_bought >1;
```

## Section 4 — Subqueries & CTEs (5 questions)
31. Find users whose total spend is greater than the average spend.
```sql
SELECT * FROM (
  SELECT u.id, u.name, SUM(o.quantity*p.price) AS total_spent
  FROM users u JOIN orders o ON u.id=o.user_id
  JOIN products p ON o.product_id=p.id
  GROUP BY u.id
) t
WHERE total_spent > (SELECT AVG(total_spent) FROM (
  SELECT SUM(o.quantity*p.price) AS total_spent
  FROM orders o JOIN products p ON o.product_id=p.id
  GROUP BY o.user_id
) x);
```
32. List products priced above the average price of all products.
```sql
SELECT * FROM products WHERE price > (SELECT AVG(price) FROM products);
```
33. Find the top-spending user per category.
```sql
WITH user_category_spend AS (
  SELECT o.user_id, p.category, SUM(o.quantity*p.price) AS spend
  FROM orders o JOIN products p ON o.product_id=p.id
  GROUP BY o.user_id, p.category
)
SELECT category, user_id, spend
FROM (
  SELECT category, user_id, spend, RANK() OVER(PARTITION BY category ORDER BY spend DESC) AS rnk
  FROM user_category_spend
) t
WHERE rnk=1;
```
34. List orders where quantity is greater than the average quantity.
```sql
SELECT * FROM orders WHERE quantity > (SELECT AVG(quantity) FROM orders);
```
35. Find the second highest priced product.
```sql
SELECT * FROM products ORDER BY price DESC LIMIT 1 OFFSET 1;
```

## Section 5 — Window Functions (5 questions)
36. Rank users by total spend.
```sql
SELECT user_id, total_spent, RANK() OVER (ORDER BY total_spent DESC) AS rnk
FROM (
  SELECT user_id, SUM(o.quantity*p.price) AS total_spent
  FROM orders o JOIN products p ON o.product_id=p.id
  GROUP BY user_id
) t;
```
37. Calculate cumulative revenue by order date.
```sql
SELECT order_date, SUM(o.quantity*p.price) AS daily_revenue, SUM(SUM(o.quantity*p.price)) OVER (ORDER BY order_date) AS cumulative_revenue
FROM orders o JOIN products p ON o.product_id=p.id
GROUP BY order_date;
```
38. Find running total of orders for each user.
```sql
SELECT user_id, order_date, SUM(quantity) OVER (PARTITION BY user_id ORDER BY order_date) AS running_total_qty FROM orders;
```
39. Show top 3 products per category based on revenue.
```sql
WITH product_revenue AS (
  SELECT p.category, p.id AS product_id, SUM(o.quantity*p.price) AS revenue
  FROM orders o JOIN products p ON o.product_id=p.id
  GROUP BY p.id
)
SELECT category, product_id, revenue
FROM (
  SELECT *, RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rnk FROM product_revenue
) t
WHERE rnk<=3;
```
40. Compute average order value per user and compare with overall average.
```sql
SELECT user_id, AVG(total) AS user_avg, (SELECT AVG(quantity*price) FROM orders o JOIN products p ON o.product_id=p.id) AS overall_avg
FROM orders o JOIN products p ON o.product_id=p.id
GROUP BY user_id;
```

## Section 6 — Advanced (5 questions)
41. Find users with increasing monthly spend over the last 3 months.
```sql
WITH monthly_spend AS (
  SELECT user_id, DATE_FORMAT(order_date,'%Y-%m') AS month, SUM(quantity*price) AS spend
  FROM orders o JOIN products p ON o.product_id=p.id
  GROUP BY user_id, month
)
SELECT user_id
FROM (
  SELECT *, LEAD(spend,1) OVER(PARTITION BY user_id ORDER BY month) AS next_month, LEAD(spend,2) OVER(PARTITION BY user_id ORDER BY month) AS next2_month
  FROM monthly_spend
) t
WHERE spend < next_month AND next_month < next2_month;
```
42. Detect duplicate orders (same user, same product, same date).
```sql
SELECT user_id, product_id, order_date, COUNT(*) AS duplicates
FROM orders
GROUP BY user_id, product_id, order_date
HAVING duplicates>1;
```
43. Find inactive users (signed up but never ordered).
```sql
SELECT u.id, u.name FROM users u
LEFT JOIN orders o ON u.id=o.user_id
WHERE o.id IS NULL;
```
44. Calculate cohort retention per signup month.
```sql
WITH first_order AS (
  SELECT user_id, DATE_FORMAT(MIN(order_date),'%Y-%m') AS cohort_month FROM orders GROUP BY user_id
), user_orders AS (
  SELECT user_id, DATE_FORMAT(order_date,'%Y-%m') AS order_month FROM orders
)
SELECT f.cohort_month, u.order_month, COUNT(DISTINCT u.user_id) AS active_users
FROM first_order f JOIN user_orders u ON f.user_id = u.user_id
GROUP BY f.cohort_month, u.order_month
ORDER BY f.cohort_month, u.order_month;
```
45. Identify top 5 categories contributing to revenue growth.
```sql
WITH revenue_growth AS (
  SELECT category, SUM(quantity*price) AS revenue
  FROM orders o JOIN products p ON o.product_id=p.id
  WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
  GROUP BY category
)
SELECT category, revenue FROM revenue_growth ORDER BY revenue DESC LIMIT 5;
```

## Section 7 — Data Cleaning & Transformation (5 questions)
46. Replace missing product categories with 'Unknown'.
```sql
UPDATE products SET category='Unknown' WHERE category IS NULL;
```
47. Convert order_date strings to DATE format.
```sql
UPDATE orders SET order_date=STR_TO_DATE(order_date, '%Y-%m-%d') WHERE order_date IS NOT NULL;
```
48. Standardize email addresses to lowercase.
```sql
UPDATE users SET email=LOWER(email);
```
49. Remove orders with NULL quantity.
```sql
DELETE FROM orders WHERE quantity IS NULL;
```
50. Aggregate multiple small categories into 'Other'.
```sql
UPDATE products SET category='Other' WHERE category IN ('Accessories','Beauty') AND id>0; -- Example mapping
```

---

All 50 questions now include practical SQL solutions ready to execute on your ecommerce practice dataset.

###william_fazle
