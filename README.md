# :ramen: Case Study #1: Danny's Diner


## :books: Table of Contents

- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

This case study is based on information from the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Question and Solution 
Please Note: I've used Oracle SQL Developer.

1:- What is the total amount each customer spent at the restaurant?

        Select SALES.CUSTOMER_ID, sum(MENU.PRICE) total_amount
        from SALES  
        inner join MENU 
        ON SALES.PRODUCT_ID = MENU.PRODUCT_ID
        group by SALES.CUSTOMER_ID
        order by SALES.CUSTOMER_ID asc;
