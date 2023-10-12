# :ramen: Case Study #1: Danny's Diner
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## :books: Table of Contents

- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

This case study is based on information from the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

***
## Please Note: I've used Oracle SQL Developer and I've made changes in the schema.

## Table Creations

        CREATE TABLE 
                customer (
                customer_id Number PRIMARY KEY,
                Join_date TIMESTAMP
                        );


        CREATE TABLE menu (
                  product_id Number PRIMARY KEY,
                  product_name VARCHAR(50),
                  price INTEGER
                        );


        CREATE TABLE Sales (
                  customer_id Number,
                  order_date TIMESTAMP,
                  product_id Number,
                  FOREIGN KEY(customer_id) REFERENCES customer (customer_id),
                  FOREIGN KEY(product_id) REFERENCES menu(product_id)
                        );

## Insert data into the tables

## Customer Table:

        INSERT INTO customer (customer_id, Join_date) VALUES (3, date '2021-01-10');

## Menu Table

        INSERT INTO menu (product_id, product_name, price) VALUES (1, 'Sushi', 10);
        
        INSERT INTO menu (product_id, product_name, price) VALUES (2, 'Curry', 15);
        
        INSERT INTO menu (product_id, product_name, price) VALUES (3, 'Ramen', 12);


## Sales Table

        Insert INTO sales (customer_id, order_date, product_id) VALUES (1, date '2021-01-01', 1);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (1, date '2021-01-01', 2);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (1, date '2021-01-07', 2);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (1, date '2021-01-10', 3);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES  (1, date '2021-01-11', 3);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES  (1, date '2021-01-11', 3);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (2, date '2021-01-01', 2);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (2, date '2021-01-02', 2);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (2, date '2021-01-04', 1);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (2, date '2021-01-11', 1);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES  (2, date '2021-01-16', 3);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (2, date '2021-02-01', 3);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES  (3, date '2021-01-01', 3);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (3, date '2021-01-01', 3);
        
        Insert INTO sales (customer_id, order_date, product_id) VALUES (3, date '2021-01-07', 3);

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Question and Solution 

1:- What is the total amount each customer spent at the restaurant?

        Select 
        SALES.CUSTOMER_ID, sum(MENU.PRICE) as total_amount
        from SALES  
        inner join MENU 
        ON SALES.PRODUCT_ID = MENU.PRODUCT_ID
        group by SALES.CUSTOMER_ID
        order by SALES.CUSTOMER_ID asc; 


#### Answer:
| customer_id | total_amount |
| ----------- | ------------ |
| 1           | 91           |
| 2           | 74           |
| 3           | 36           |


***


**2. How many days has each customer visited the restaurant?**

        SELECT 
        customer_id, 
        COUNT(DISTINCT order_date) AS visit_count
        FROM dannys_diner.sales
        GROUP BY customer_id;


#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

***


**3. What was the first item from the menu purchased by each customer? **

        With Orders as (
        Select SALES.CUSTOMER_ID as Customer, Menu.Product_Name as Product, SALES.ORDER_DATE,
        DENSE_RANK() OVER (PARTITION BY SALES.CUSTOMER_ID ORDER BY SALES.ORDER_DATE ) as Rank
        from SALES  
        Join Menu
        ON SALES.PRODUCT_ID = MENU.PRODUCT_ID
        )
        Select Customer, Product from Orders
        where Rank = 1;


#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| 1           | curry        | 
| 1           | sushi        | 
| 2           | curry        | 
| 3           | ramen        |
| 3           | ramen        |

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

        Select count(SALES.PRODUCT_ID) as Most_Sold_Product, Menu.Product_Name 
        from SALES  
        Join Menu
        ON SALES.PRODUCT_ID = MENU.PRODUCT_ID
        group by Menu.Product_Name
        order by Most_Sold_Product desc
        limit 1;


#### Answer:
| most_purchased | product_name | 
| ----------- | --------------- |
| 8           | ramen           |

***

**5. Which item was the most popular for each customer?**

        With Customer_Popular as (
        Select count(SALES.PRODUCT_ID) as More_Sold_Product, SALES.CUSTOMER_ID as Customer , Menu.Product_Name as Product_Name,
        DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(sales.customer_id) DESC) AS rank
        from SALES  
        Join Menu
        ON SALES.PRODUCT_ID = MENU.PRODUCT_ID
        group by SALES.CUSTOMER_ID, Menu.Product_Name
        order by More_Sold_Product desc)
        
        Select Customer,Product_Name, More_Sold_Product
        from Customer_Popular
        where Rank = 1
        ;


#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| 1           | ramen        |  3   |
| 1           | Curry        |  3   |
| 2           | sushi        |  2   |
| 2           | curry        |  2   |
| 2           | ramen        |  2   |
| 3           | ramen        |  3   |

***

**6. Which item was purchased first by the customer after they became a member?**





































