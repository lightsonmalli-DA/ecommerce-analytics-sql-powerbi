Create database E_comm;
Use E_comm;
select * from customers_v2;
select * from order_items_v2;
select * from orders_v2;
select * from products_v2;
# Daily Sales and Revenue
# Step 1: Change Data Type
Select str_to_date(order_date and delivery_date, "%Y-%m-%d") 
from orders_v2;
select * from orders_v2;
# Step 2 Create view for Daily sales revenue
create view daily_sales_revenue as
Select order_date, round(sum(selling_price*quantity),2) as Revenue
from order_items_v2
join orders_v2 using (order_id)
group by 1
order by order_date;

Select * from daily_sales_revenue;
# Top 10 selling products
Create view Top_10_Product as 
Select product_name, Total_QTY, Revenue, 
rank() over (order by Revenue desc) as Rev_Rank
from
(
Select p.product_name,sum(oi.quantity) as Total_QTY, round(sum(oi.selling_price*oi.quantity),2) as Revenue
from products_v2 p
inner join order_items_v2 oi 
on p.product_id = oi.product_id
group by 1) top
order by Revenue desc;

Select product_name, Total_QTY, Revenue
from Top_10_Product
limit 10;
# Customer Repeat Rate

Create view CRR as
with cte as (
select c.customer_name, count(o.customer_id) as order_count
from customers_v2 c
join orders_v2 o
on o.customer_id= c.customer_id
group by 1)
Select round(sum(order_count>=2)*1.0/count(*)*100,2) as Repeat_Percentage
from cte;

Select * from CRR;

#Order Delivery performance
Create View ODR as
select order_status,
round(count(order_id)/(Select count(*) from orders_v2)*100,2) as ODP
from orders_v2
group by 1;

Select * from ODR;

# M-O-M growth
Create view MOM_growth as
With mom as (
select date_format(order_date,"%Y-%m-01") as month, 
round(sum(selling_price*quantity),2) as Rev
from orders_v2
join order_items_v2 using(order_id)
group by 1 )
Select month, round(((Rev-lag(Rev) over (order by month))/(lag(Rev)over (order by month)))*100,2) as MOM_growth
from mom;

Select * from MOM_growth
order by month asc;
# AOV
Create view AOV as
Select round(sum(quantity*selling_price)/count(order_id),2) as AOV_Rev 
from order_items_v2;

# CLV
create view CLV as
select customer_name, Orders, C_Rev, 
timestampdiff(day,First_Order,Last_order) as CLS,
round((C_Rev/Orders)*(orders/timestampdiff(day,First_Order,Last_order))*timestampdiff(day,First_Order,Last_order),2) as Customer_lifetime_Value
from
(
Select customer_name,count(order_id) as Orders,round(sum(quantity*selling_price),2) as C_Rev,
min(order_date) as First_Order,
max(order_date) as Last_order
from order_items_v2
join orders_v2 using (order_id)
join customers_v2 using (customer_id)
group by 1) Life
order by Customer_lifetime_Value desc;

Select * from CLV;
