
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
* Retrieves data from the 'sales' and 'menu' tables and combines them using a JOIN operation based on the 'product_id'.
* Calculates the total sales for each member using the SUM function, and groups the results by member.
* Sorts the result set in descending order of total sales (from highest to lowest).

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
* Uses the COUNT function with DISTINCT to calculate the number of distinct days a customer has visited the diner and it groups the results by customer.
  
#### Answer
- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

### 3. What was the first item from the menu purchased by each customer?
```

```
#### Code Explanation
#### Answer

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```

```
#### Code Explanation
#### Answer

### 5. Which item was the most popular for each customer?
```

```
#### Code Explanation
#### Answer

### 6. Which item was purchased first by the customer after they became a member?
```

```
#### Code Explanation
#### Answer

### 7. Which item was purchased just before the customer became a member?
```

```
#### Code Explanation
#### Answer

### 8. What is the total items and amount spent for each member before they became a member?
```

```
#### Code Explanation
#### Answer

### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```

```
#### Code Explanation
#### Answer

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```

```
#### Code Explanation
#### Answer

