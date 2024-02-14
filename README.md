Pizza Store SQL Project - Guide

Objectives:
	- Build a SQL Database
	- Write custom SQL queries



Scenario:
	The client is opening new pizza store for delivery.

Project brief:
	Design and build relational database for his business to allow him to capture and store all the important informations and data performance that the business generates. This will help client to monitor business performance. We are taking care only for back-end. He is hiring somebody else to build the front-end ordering system. Three brief areas that requires us to concentrate on:
	1. Customer orders
	2. Stock control
	3. Staff




1. CUSTOMER ORDERS - Data requirements
-----------------------------
Specifying all the fields for the data we want to collect.
Normalizing the data - this mean organizing the data to ensure it appears similar across all fields and records. Normalizing also means adding more realted tables and defining the tables relationships.

Data that our client want to collect for each order is:
	- Item name
	- Item price
	- Quantity
	- Customer name
	- Delivery address
This is starting point for us, to specify all of the fields we'll need for our "Orders table". We will need to add "Order ID" for example and also to split "Delivery address" into different parts.
From looking at the Menu that the clien has provided to us, we can see that there are different sizes of pizza and beverages, so we can include this as a separate field. We could also include a field for the product category (pizza, desserts, beverages).
Along the way, we are creating initial Excel table, with the columns we will need. We are adding "row_id" column, because it will serve us as a primary key. We cannot user "order_id" as a primary key because one order probably will have multiple items on it.
Lets break down what is client table of data containing and what we think should contain our table:

Client table of data >  Item name				Our table of data > Row ID
			Item price						    Order ID
			Quantity						    Item name
			Customer name						    Item category
			Delivery address					    Item size
										    Item price
										    Quantity
										    Customer first name
										    Customer last name
										    Delivery address 1
										    Delivery address 2
										    Delivery city
										    Delivery zipcode


For designing and building our new data base, we are going to use QUICK DATABESE DIAGRAMS.

We are noticing that when there is an order with multiple items on it - there is redundancy (multiple copies of the same information are stored in more than one place at a time). We manage this with the process of normalization. Normalization does 2 things: reduce redundancy and improves efficiency.
In our case, to deal with redundancy, we are going to create additional tables for both "customers" and "delivery addresses". We will use "Identifer" for this data in our "Order table". It will make our database much more efficient.
Now, "cust_id" has been specified as a foreign key connected to "order.cust_id". We adding "NULL" constraint to "delivery_address2" field, because QUCIK DATABASE DIAGRAMS by default applies "NOT NULL" constraint to every single field, meaning that it cannot contain NULL values. However, field "delivery_address2" is not always needed, hence in fact can be NULL.
Next, we need to create "Item table". It will reduce the amount of the data in "Order table" and second, more important, if client want to change the name of some products, he doesn't need to go through every single order to edit that, but can do only once in the "Item table".

------------------------------------------------
>>> SQL queries for first area - CUSTOMER ORDERS
------------------------------------------------
	* Total orders
	* Total sales
	* Total items sold
	* Average order value
	* Sales by category
	* Top selling items
	* Orders by hour
	* Sales by hour
	* Orders nu address
	* Orders by delivery method





>>SQL Query :

SELECT 
	o.order_id,
	i.item_price,
	o.quantity,
	i.item_cat,
	i.item_name,
	o.created_at,
	a.delivery_address1,
	a.delivery_address2,
	a.delivery_city,
	a.delivery_zipcode,
	o.delivery
FROM 
	orders AS o
	LEFT JOIN item AS i 
	ON o.item_id = i.item_id
	LEFT JOIN address AS a 
	ON o.add_id = a.add_id








2. STOCK CONTROLL - Data requirements
-----------------------------
The client wants to be able to know when it's time to order new stock. To do this, we are going to need more informations about:
	- what ingredients go into each pizza
	- their quantity based on the size of the pizza
	- the existing stock level

We'll assume that the lead time for delivery by suppliers is the same for all ingredients.
The client gave us all the necessary infromations on each pizza ingredient and the weight that the item is sold in. He also gave us a list of ingredients and the amounts that go into each pizza. These 2 tables are "Recipe" and "Ingredient" table. We will create these 2 tables in QUICK DATABESE DIAGRAMS also. We can see from the data that the "recipe_ID" is the same as the "sku" from the "Item" table. And "ingredient_id" appears in both "Ingredient" and "Recipe" table - so those are the relationships. 
With all of this data, the client will be able to calculate exactly how much each pizza cost to make. If supplly prices go up - he will just need to update the ingredient prices in the "Ingredient" table.
Finally, we need a table to hold stock level for each ingredient. "Inventory" table is the solution, which will contain "inventory_id", "item_id" and "quantity".







>>SQL Query :

More complicated than the first one. Mainly because we need to calculate how much inventory we're using and then indentify inventory that needs reordering. We also want to calculate how much each pizza costs to make, based on the cost of the ingredients so we can keep an eye on pricing and P/L. Here's what we need: 
	- Total quantity by ingredient
	- Total cost of ingredients
	- Calculated cost of pizza
	- Percentage stock remaining by ingredient
	- List of ingredients to reorder, based on remaining inventory
We are going to need to create two different VIEWS of the data in order to give us the result we need.
Calculated cost of pizza can be messured as Number of orders * Ingredients quantity in recipe. First, we need to create "Order quantity per pizza". Than, we need to break down these pizzas by ingredients that come from the "Recipe" table - so we need to JOIN this.
Next, we need to calculate the total cost of ingredient.
Then, we meed to multiply "order_quantity" by "recipe_quantity". But, because "order_quantity" is already calculated field, we cannot use it in another calculation in the SELECT statement. The solution here is called SUBQUERIES.
For further manipulations, we need to turn our custom query to new VIEW. If we want to work with data in this VIEW, we just need to reffer to it.
Now, we want to calculate:
	- Total weight ordered
	- Inventory amount
	- Inventory remaining per ingredient
First step is to aggregate the ordered weight for each ingredient. Next, we need the total weight of inventory for each ingredient.
		Total weight in stock = ingredient quantity * ingredient weight
Lastly, we need to calculate total inventory weight minus the ordered weight as "remaining_weight".


CREATE VIEW stock2 AS
SELECT 
	s1.item_name,
	s1.ing_id,
	s1.ing_name,
	s1.ing_weight,
	s1.ing_price,
	s1.order_quantity,
	s1.recipe_quantity,
	s1.order_quantity*s1.recipe_quantity AS ordered_weight,
	s1.ing_price/s1.ing_weight AS unit_cost,
	(s1.order_quantity*s1.recipe_quantity)*(s1.ing_price/s1.ing_weight) AS ingredient_cost
FROM 
	(SELECT 
	o.item_id,
	i.sku,
	i.item_name,
	r.ing_id,
	ing.ing_name,
	r.quantity AS recipe_quantity,
	SUM(o.quantity) AS order_quantity,
	ing.ing_weight,
	ing.ing_price
FROM orders AS o
LEFT JOIN item AS i
	ON o.item_id = i.item_id
LEFT JOIN recipe AS r
	ON i.sku = r.recipe_id
LEFT JOIN ingredient AS ing 
	ON ing.ing_id = r.ing_id
GROUP BY 
	o.item_id,
	i.sku,
	i.item_name,
	r.ing_id,
	r.quantity,
	ing.ing_name,
	ing.ing_weight,
	ing.ing_price) AS s1


SELECT *
FROM stock2
 

SELECT 
	s2.ing_name,
	s2.ordered_weight,
	ing.ing_weight*inv.quantity AS total_inv_weight,
	(ing.ing_weight*inv.quantity)-s2.ordered_weight AS remaining_weight
FROM (
	SELECT 
	ing_id,
	ing_name,
	SUM(ordered_weight) AS ordered_weight
FROM stock2
GROUP BY ing_name, ing_id) AS s2
LEFT JOIN inventory AS inv 
	ON inv.item_id = s2.ing_id
LEFT JOIN ingredient AS ing 
	ON ing.ing_id = s2.ing_id








3. STAFF - Data requirements
-----------------------------
Which staff members are working when
Based on the staff salary informations, how much each pizza costs(ingredients + chefs + delivery)

Here we have "Staff" table and "Shift" table. What we are going to need is "Rota" table - which will tell us who is working when. For this, we'll need:
	- row_id
	- rota_id
	- date
	- shift_id
	- staff_id

Connections between these 3 tables are:
	- shift_id ("Shift" table)  >  shift_id ("Rota" table)
	- staff_id ("Staff" table)  >  staff_id ("Rota" table)
	- date ("Rota" table)       >  created_at ("Order" table)


>>SQL Query :


After all JOINS here, we need to calculate staff cost per role. We will use DATEDIFF() method, turning the result into total number of minutes and then deviding this by 60, to get number of hours as a decimal to then be multiplied by our hourly rate.






SELECT
	rota.date,
	s.first_name,
	s.last_name,
	s.hourly_rate,
	DATEDIFF(minute, sh.start_time, sh.end_time) AS minutes_in_shift,
	ROUND((DATEDIFF(minute, sh.start_time, sh.end_time) * s.hourly_rate)/60,2) AS staff_cost
FROM rota
LEFT JOIN staff AS s 
	ON rota.staff_id = s.staff_id
LEFT JOIN shift AS sh
	ON sh.shift_id = rota.shift_id




