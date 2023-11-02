
 # :ramen: Case Study #1: Danny's Diner
 <img width="741" alt="Screenshot 2023-10-24 at 6 54 19 PM" src="https://github.com/ByThaoNguyen/8-Week-SQL-Challenge/assets/116039570/793f8669-6b3a-4fc6-bec7-0bbba373795b">

## :scroll: Table of Contents
* Business Task
* Entity Relationship Diagram
* Creating database schema
* Questions and Answer
  
## Business Task
In 2021, Danny opened Danny's Diner, a restaurant specializing in sushi, curry, and ramen. He has collected data on customer behavior but need help analyzing it. Danny aims to use this data to understand customer patterns, spending, and menu preferences, enhancing customer experiences and potentially expanding their loyalty program. 

## Entity Relationship Diagram
Due to privacy concerns, Danny only shared a sample of his comprehensive customer data, with the hope that these examples will suffice to create fully functional SQL queries to help him answer his questions.

The data set contains Sales, Members, and Menu tables. Please refer to the relationship diagram below to understand the connection.

<img width="725" alt="Screenshot 2023-10-24 at 7 20 12 PM" src="https://github.com/ByThaoNguyen/8-Week-SQL-Challenge/assets/116039570/7855360b-75a3-4592-af77-ecde6f850c0c">

## Creating Database Schema

```
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```


## Questions and Answers
### 1. What is the total amount each customer spent at the restaurant?
```
SELECT
  s.customer_id,
  sum(m.price) AS total_sales
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m
  ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY total_sales DESC
```

#### Code Explanation
* Retrieve data from the 'sales' and 'menu' tables and combine them using a JOIN operation based on the 'product_id'.
* Calculate the total sales for each member using the SUM function, and groups the results by member.
* Sort the result set in descending order of total sales (from highest to lowest).

#### Answer
- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

### 2. How many days has each customer visited the restaurant?
```
SELECT
  s.customer_id,
  COUNT (DISTINCT s.order_date) as number_days_visit
FROM dannys_diner.sales as s
GROUP BY s.customer_id
```
#### Code Explanation
* Use the COUNT function with DISTINCT to calculate the number of distinct days a customer has visited the diner and it groups the results by customer.
  
#### Answer
- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

### 3. What was the first item from the menu purchased by each customer?
```
WITH sales_by_order AS (
  SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT 
  customer_id, 
  product_name
FROM sales_by_order
WHERE rank = 1
GROUP BY customer_id, product_name;
```
#### Code Explanation
* Create a Common Table Expression (CTE) called sales_by_order. Since order_date doesn't have a timestamp, it's hard to determine the exact sequence of the customer's orders. Therefore, I used the dense_rank() window function in the CTE to create a rank column; orders in the same day will have the same rank.
* Select the appropriate columns with conditions in the outer query and group the results with GROUP BY clause.
  
#### Answer
- Customer A’s first order are curry and sushi.
- Customer B’s first order is curry.
- Customer C’s first order is ramen.

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```
SELECT 
  menu.product_name,
  COUNT(sales.product_id) AS times_purchased
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY times_purchased DESC
LIMIT 1;
```
#### Code Explanation
- Use COUNT aggregation on product_id column to count the number times of purchased for each product in product_id column and ORDER BY the result in descending order.
- Use LIMIT to get the most purchased product.
  
#### Answer
- The most purchased item on the menu is ramen which is 8 times.

### 5. Which item was the most popular for each customer?
```
WITH most_popular AS (
  SELECT 
    sales.customer_id, 
    menu.product_name, 
    COUNT(menu.product_id) AS order_count,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id 
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.menu
  JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT 
  customer_id, 
  product_name, 
  order_count
FROM most_popular 
WHERE rank = 1;
```

#### Code Explanation
- Create a Common Table Expression (CTE) called most_popular and within the CTE, join the menu table and sales table using the product_id column.
- Group results by sales.customer_id and menu.product_name.
- Calculate the count of menu.product_id occurrences for each group.
- Use DENSE_RANK() window function to calculate the ranking of each sales.customer_id partition based on the count of orders COUNT(sales.customer_id) in descending order.
- Select the appropriate columns and apply a filter in the WHERE clause to retrieve only the rows where the rank column equals 1 which has the highest order count for each customer in the outer query.
  
#### Answer
- Customer A like ramen the most with 3 times order
- Customer B like all items on the menu with the same 2 times order for each item
- Customer C like ramen the most with 3 times order

### 6. Which item was purchased first by the customer after they became a member?
```
WITH orders_after_became_members AS (
  SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  Where sales. order_date >= members.join_date
)

SELECT 
  customer_id, 
  product_name
FROM orders_after_became_members
WHERE rank = 1
GROUP BY customer_id, product_name;

```
#### Code Explanation
- Create a Common Table Expression (CTE) called orders_after_became_members.
- Within the CTE, join the sales table and menu table using the product_id column; join the sales table and members table using the customer_id column. Additionally, apply a condition to include only sales that occurred on or after the member's join_date (sales.order_date > = members.join_date).
- Select the appropriate columns and calculate the row number using the ROW_NUMBER() window function. The PARTITION BY clause divides the data by members.customer_id and the ORDER BY clause orders the rows within each members.customer_id partition by sales.order_date.
- Select the appropriate columns and apply a filter in the WHERE clause to retrieve only the rows where the rank column equals 1 which has the highest order count for each customer in the outer query.

#### Answer
- Customer A's first order as a member is ramen.
- Customer B's first order as a member is sushi.

### 7. Which item was purchased just before the customer became a member?
```
WITH order_before_become_member AS(
  SELECT 
    sales.customer_id,
    sales.order_date,
    menu.product_name 
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  WHERE sales. order_date < members.join_date
)

SELECT customer_id,
       product_name
FROM order_before_become_member
GROUP BY customer_id, product_name
```
#### Code Explanation
- Create a Common Table Expression (CTE) called order_before_become_member.
- Within the CTE, join the sales table and menu table using the product_id column; join the sales table and members table using the customer_id column. Additionally, apply a condition to include only sales that occurred before the member's join_date (sales.order_date < members.join_date). Then select the appropriate columns.
- Select the appropriate columns in the outer query and group them by customer_id and product_name.

#### Answer
- Both customers A and B ordered curry and sushi before became members.

<img width="389" alt="Screenshot 2023-11-01 at 12 42 45 AM" src="https://github.com/ByThaoNguyen/8-Week-SQL-Challenge/assets/116039570/f8f47a09-b4ef-4452-9f13-a31846babdd7">


### 8. What is the total items and amount spent for each member before they became a member?
```
SELECT 
  sales.customer_id, 
  COUNT(sales.product_id) AS total_items, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  AND sales.order_date < members.join_date
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
```
#### Code Explanation
- Select the columns sales.customer_id, calculate the count of sales.product_id as total_items for each customer, and sum the menu.price as total_sales.
- Join sales tables and members table on customer_id column, ensuring that sales.order_date is earlier than members.join_date (sales.order_date < members.join_date). Then, join the menu table with the sales table on product_id column.
- Group them by sales.customer_id.

#### Answer
- Customer A spent $25 on 2 items before becoming a member.
- Customer B spent $40 on 3 items before becoming a member.

<img width="439" alt="Screenshot 2023-11-01 at 12 54 34 AM" src="https://github.com/ByThaoNguyen/8-Week-SQL-Challenge/assets/116039570/7484c8cc-e819-4daf-a9d1-d74eaca772ec">

### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```
WITH points_cte AS (
  SELECT 
    menu.product_id, 
    CASE
      WHEN product_id = 1 THEN price * 20
      ELSE price * 10 END AS points
  FROM dannys_diner.menu
)

SELECT 
  sales.customer_id, 
  SUM(points_cte.points) AS total_points
FROM dannys_diner.sales
INNER JOIN points_cte
  ON sales.product_id = points_cte.product_id
GROUP BY sales.customer_id
```
#### Code Explanation
- Create a a Common Table Expression (CTE) called points_cte.
- Within the CTE, caculate points customer will get if they order sushi, curry, or ramen.
- In the outer query, select the appropriate columns, join with points_cte, and group them by customer_id.

#### Answer
- Total points for Customer A is $860.
- Total points for Customer B is $940.
- Total points for Customer C is $360.

<img width="434" alt="Screenshot 2023-11-01 at 1 09 00 AM" src="https://github.com/ByThaoNguyen/8-Week-SQL-Challenge/assets/116039570/062d72b9-7c72-43d7-ad70-f65c86c7bcc9">

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```
WITH dates_cte AS (
  SELECT 
    m.customer_id, 
    m.join_date, 
    m.join_date + INTERVAL '6 days' AS valid_date, 
    DATE_TRUNC('month', '2021-01-31'::DATE) + INTERVAL '1 month' - INTERVAL '1 day' AS last_date
  FROM dannys_diner.members AS m
)

SELECT 
  s.customer_id, 
  SUM(
    CASE
      WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price
      WHEN s.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 * menu.price
      ELSE 10 * menu.price
    END
  ) AS points
FROM dannys_diner.sales AS s
INNER JOIN dates_cte AS dates
  ON s.customer_id = dates.customer_id
  AND dates.join_date <= s.order_date
  AND s.order_date <= dates.last_date
INNER JOIN dannys_diner.menu AS menu
  ON s.product_id = menu.product_id
GROUP BY s.customer_id;

```
PS: Assumpting the logic in question 9 still applying 

#### Code Explanation
- In the "dates_cte," calculate the "valid_date" by adding 6 days to the "join_date" and determine the "last_date" of the month by subtracting 1 day from the last day of January 2021.
- From the "dannys_diner.sales" table, join the "dates_cte" using the "customer_id" column, ensuring that the "order_date" of the sale is not later than the "last_date" (sales.order_date <= dates.last_date).
- Then, join the "dannys_diner.menu" table based on the "product_id" column.
- In the outer query, calculate the "points" by using a CASE statement to determine the points based on the following assumptions: If the "product_name" is 'sushi', multiply the price by 2 and then by 10. For orders placed between "join_date" and "valid_date," also multiply the price by 2 and then by 10. For all other products, multiply the price by 10.
- Calculate the sum of points for each customer.

#### Answer
- Customer A has 1,370 points.
- Customer B has 820 points.
  
<img width="369" alt="Screenshot 2023-11-01 at 10 29 01 PM" src="https://github.com/ByThaoNguyen/8-Week-SQL-Challenge/assets/116039570/205d1612-7477-4d87-90bd-b0b0c63aab6b">

