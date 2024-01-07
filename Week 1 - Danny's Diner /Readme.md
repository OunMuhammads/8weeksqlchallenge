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
## Note: I've made some small tweaks to the database schema. I used MySQL, but feel free to apply the changes using your preferred tool.

## Table Creations

        CREATE TABLE 
            Members (
            customer_id int PRIMARY KEY,
            Join_date TIMESTAMP
                    );


    CREATE TABLE Menu (
              product_id int PRIMARY KEY,
              product_name VARCHAR(50),
              price INTEGER
                    );


    CREATE TABLE Sales (
              customer_id int,
              order_date TIMESTAMP,
              product_id int,
              FOREIGN KEY(customer_id) REFERENCES members (customer_id),
              FOREIGN KEY(product_id) REFERENCES menu (product_id)
                    );
                    
## Insert data into the tables

## Members Table:

	INSERT INTO members
	          (customer_id, join_date)
	VALUES
	          ('1', '2021-01-07'),
	          ('2', '2021-01-09'),
	          ('3', '2021-01-10');

## Menu Table

	INSERT INTO menu 
		(product_id, product_name, price) 
	VALUES 
		(1, 'Sushi', 10),
		(2, 'Curry', 15),
		(3, 'Ramen', 12);


## Sales Table

	Insert INTO sales 
		(customer_id, order_date, product_id) 
	VALUES 
		(1, date '2021-01-01', 1),
		(1, date '2021-01-01', 2),
		(1, date '2021-01-07', 2),
		(1, date '2021-01-10', 3),
		(1, date '2021-01-11', 3),
		(1, date '2021-01-11', 3),
		(2, date '2021-01-01', 2),
		(2, date '2021-01-02', 2),
		(2, date '2021-01-04', 1),
		(2, date '2021-01-11', 1),
		(2, date '2021-01-16', 3),
		(2, date '2021-02-01', 3),
		(3, date '2021-01-01', 3),
		(3, date '2021-01-01', 3),
		(3, date '2021-01-07', 3);
  
## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Question and Solution 


**1:- What is the total amount each customer spent at the restaurant?**

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
| 1           | 76           |
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
| 1           | 4           |
| 2           | 6           |
| 3           | 2           |

***


**3. What was the first item from the menu purchased by each customer?**

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
| ----------- | ------------ |
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
| -------------- | -------------|
| 8              | ramen        |

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
| ----------- | ------------ |------------ |
| 1           | ramen        |  3          |
| 3           | ramen        |  3          |
| 2           | sushi        |  2          |
| 2           | curry        |  2          |
| 2           | ramen        |  2          |

***

**6. Which item was purchased first by the customer after they became a member?**

	With CTE AS (
		Select s.customer_id, s.order_date, s.Product_id,
		Row_Number() Over (Partition by customer_id order by order_date) as Row_num
		from members as m
		join Sales as s
		on m.customer_id = s.customer_id
		where s.order_date > m.join_date
		    )
	Select c.customer_id,m.product_name
	from CTE as c
	join Menu as m
	on c.product_id = m.product_id
	where Row_num = 1
	order by customer_id asc
 	;


#### Answer:
| customer_id | product_name | 
| ----------- | ------------ |
| 1           | Ramen        |  
| 2           | Sushi        |


***

**7. Which item was purchased just before the customer became a member?**

	With CTE AS (
		Select s.customer_id, s.order_date, s.Product_id,
		Row_Number() Over (Partition by customer_id order by order_date desc) as Row_num
		from members as m
		join Sales as s
		on m.customer_id = s.customer_id
		where s.order_date < m.join_date
		    )
	Select c.customer_id,m.product_name
	from CTE as c
	join Menu as m
	on c.product_id = m.product_id
	where Row_num = 1
	order by customer_id asc;


#### Answer:
| customer_id | product_name | 
| ----------- | ------------ |
| 1           | Curry        |  
| 2           | Sushi        |  
| 3           | Ramen        |


***

**8. What is the total items and amount spent for each member before they became a member?**

	Select s.customer_id,  
	count(s.Product_id) as total_items, 
	Sum(Price) as total_sales
	from members as m
	join Sales as s
	on m.customer_id = s.customer_id
	and s.order_date < m.join_date
	join menu as n
	on n.product_id = s.product_id
	group by s.customer_id
	;


#### Answer:
| customer_id | total_items | total_sales |
| ----------- | ----------- |------------ |
| 1           | 2           |  25         |
| 2           | 3           |  40         |
| 3           | 3           |  36         |


***


**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

	With CTE AS (
	select product_id,
	Case when product_id = 1 then price * 20
	else price * 10 
	end as points
	from menu )
	select customer_id, Sum(points) as total_points
	from sales as s
	join cte as c
	on s.product_id = c.product_id
	group by customer_id;


#### Answer:
| customer_id | total_points | 
| ----------- | -----------  |
| 1           | 860          |
| 2           | 940          |
| 3           | 360          |


***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

	WITH dates_cte AS (
	  SELECT 
	    customer_id, 
	    join_date, 
	    DATE_ADD(join_date, INTERVAL 6 DAY) AS valid_date,
	    LAST_DAY('2021-01-31'- INTERVAL 1 day) AS last_date
	FROM members
	)
	SELECT 
	  sales.customer_id, 
	  SUM(CASE
	    WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price
	    WHEN sales.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 * menu.price
	    ELSE 10 * menu.price END) AS points
	FROM sales
	INNER JOIN dates_cte AS dates
	  ON sales.customer_id = dates.customer_id
	  AND dates.join_date <= sales.order_date
	  AND sales.order_date <= dates.last_date
	INNER JOIN menu
	  ON sales.product_id = menu.product_id
	GROUP BY sales.customer_id;


#### Answer:
| customer_id | points | 
| ----------- | ------ |
| 1           | 1020   |
| 2           | 320    |


***

## Bonus Questions

## Join All The Things

**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**

	SELECT 
	  s.customer_id, 
	  s.order_date,  
	  n.product_name, 
	  n.price,
	  CASE
	    WHEN m.join_date > s.order_date THEN 'N'
	    WHEN m.join_date <= s.order_date THEN 'Y'
	    ELSE 'N' END AS member_status
	FROM sales as s
	LEFT JOIN members as m
	  ON sales.customer_id = members.customer_id
	INNER JOIN menu as n
	  ON s.product_id = n.product_id
	ORDER BY m.customer_id, s.order_date;


#### Answer:
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | -----  |
| 1	      | 2021-01-01 |	sushi	  | 10	  |   N    |
| 1	      | 2021-01-01 |	curry	  | 15	  |   N    |
| 1	      | 2021-01-07 |	curry	  | 15	  |   Y    |
| 1	      | 2021-01-10 |	ramen	  | 12	  |   Y    |
| 1	      | 2021-01-11 |	ramen	  | 12	  |   Y    |
| 1	      | 2021-01-11 |	ramen	  | 12	  |   Y    |
| 2	      | 2021-01-01 |	curry	  | 15	  |   N    |
| 2	      | 2021-01-02 |	curry	  | 15	  |   N    |
| 2	      | 2021-01-04 |	sushi	  | 10	  |   N    |
| 2	      | 2021-01-11 |	sushi	  | 10	  |   Y    |
| 2	      | 2021-01-16 |	ramen	  | 12	  |   Y    |
| 2	      | 2021-02-01 |	ramen	  | 12	  |   Y    |
| 3	      | 2021-01-01 |	ramen	  | 12	  |   N    |
| 3	      | 2021-01-01 |	ramen	  | 12	  |   N    |
| 3	      | 2021-01-07 |	ramen	  | 12	  |   N    |

***

## Rank All The Things

**Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.**

	WITH customers_CTE AS (
	  SELECT 
	    s.customer_id, 
	    s.order_date,  
	    n.product_name, 
	    m.price,
	    CASE
	      WHEN m.join_date > s.order_date THEN 'N'
	      WHEN m.join_date <= s.order_date THEN 'Y'
	      ELSE 'N' END AS member_status
	  FROM sales as s
	  LEFT JOIN members as m
	    ON s.customer_id = m.customer_id
	  INNER JOIN menu as n
	    ON s.product_id = n.product_id
	)
	
	SELECT 
	  *, 
	  CASE
	    WHEN member_status = 'N' then NULL
	    ELSE RANK () OVER (
	      PARTITION BY customer_id, member_status
	      ORDER BY order_date
	  ) END AS ranking
	FROM customers_CTE;

#### Answer:
| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | -----  | ------- |
| 1	      | 2021-01-01 |	sushi	  | 10	  |   N    | NULL    |
| 1	      | 2021-01-01 |	curry	  | 15	  |   N    | NULL    |
| 1	      | 2021-01-07 |	curry	  | 15	  |   Y    | 1       |
| 1	      | 2021-01-10 |	ramen	  | 12	  |   Y    | 2       |
| 1	      | 2021-01-11 |	ramen	  | 12	  |   Y    | 3       |
| 1	      | 2021-01-11 |	ramen	  | 12	  |   Y    | 3       |
| 2	      | 2021-01-01 |	curry	  | 15	  |   N    | NULL    |
| 2	      | 2021-01-02 |	curry	  | 15	  |   N    | NULL    |
| 2	      | 2021-01-04 |	sushi	  | 10	  |   N    | NULL    |
| 2	      | 2021-01-11 |	sushi	  | 10	  |   Y    | 1       |
| 2	      | 2021-01-16 |	ramen	  | 12	  |   Y    | 2       |
| 2	      | 2021-02-01 |	ramen	  | 12	  |   Y    | 3       |
| 3	      | 2021-01-01 |	ramen	  | 12	  |   N    | NULL    |
| 3	      | 2021-01-01 |	ramen	  | 12	  |   N    | NULL    |
| 3	      | 2021-01-07 |	ramen	  | 12	  |   N    | NULL    |

***
