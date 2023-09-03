# SQL | TABLEAU
Data exploratory analysis conducted utilizing PostgreSQL and creation of a dynamic and informative dashboard in Tableau. The main goal was create a sales dashboard with the main performance indicators and the key drivers of the month's results for a car e-commerce business.<br/>

Below is my code in SQL used to conduct the Data Exploratory Analysis:<br/>

```sql
-- Project of data exploration for Car Sales Dashboard using SQL

-- (Query 1) Revenue, leads, conversion and average ticket price
-- Columns: month, leads (#), sales (#), revenue (k, R$), conversion (%), avg ticket price (k, R$)

with leads as (
	select
  		date_trunc('month', visit_page_date)::date as visit_page_month,
  		count(*) as visit_page_count
	from sales.funnel 
	group by visit_page_month 
	order by visit_page_month
	),
payments as (
	select
  		date_trunc('month', paid_date)::date as paid_month,
  		count(fun.paid_date) paid_count,
  		sum(prod.price * (1+fun.discount)) as revenue
	from sales.funnel as fun
	left join sales.products as prod
  		on fun.product_id = prod.product_id
	where fun.paid_date is not null  
	group by paid_month
	order by paid_month
	)
select
   leads.visit_page_month as "month",
   leads.visit_page_count as "leads(#)",
   payments.paid_count as "sales(#)",
   (payments.revenue/1000) as "revenue(k,R$)",
   (payments.paid_count::float/leads.visit_page_count::float) as "conversion(%)",
   (payments.revenue/payments.paid_count/1000) as "avg ticket price(k,R$)"
from leads
left join payments
   on leads.visit_page_month = payments.paid_month

-- (Query 2) Top selling states
-- Columns: country, state, sales (#)

select
	'Brazil' as country,
	cus.state,
	count (fun.paid_date) as "sales(#)"
from sales.funnel as fun
left join sales.customers as cus
	on fun.customer_id = cus.customer_id
where paid_date between '2021-08-01' and '2021-08-31'
group by country, state
order by "sales(#)" desc
limit 5

-- (Query 3) Top selling brands by month
-- Columns: brand, sales (#)

select
	pro.brand,
	count(fun.paid_date) as "sales (#)"
from sales.products as pro
left join sales.funnel as fun
	on fun.product_id = pro.product_id
where paid_date between '2021-08-01' and '2021-08-31'
group by brand
order by "sales (#)" desc
limit 5

-- (Query 4) Top selling stores
-- Columns: store, sales (#)

select
	sto.store_name as store,
	count(fun.paid_date) as "sales (#)"
from sales.funnel as fun 
left join sales.stores as sto
	on fun.store_id = sto.store_id
where paid_date between '2021-08-01' and '2021-08-31'	
group by store
order by "sales (#)" desc
limit 5


-- (Query 5) Days of week with the highest number of visits 
-- Columns: day_of_week, day, visits (#)

select
	extract('dow' from visit_page_date) as day_of_week,
	case 
		when extract('dow' from visit_page_date) = 0 then 'Sunday'
		when extract('dow' from visit_page_date) = 1 then 'Monday'
		when extract('dow' from visit_page_date) = 2 then 'Tuesday'
		when extract('dow' from visit_page_date) = 3 then 'Wednesday'
		when extract('dow' from visit_page_date) = 4 then 'Thursday'
		when extract('dow' from visit_page_date) = 5 then 'Friday'
		when extract('dow' from visit_page_date) = 6 then 'Saturday'
		else null end as "day",
	count (*) as "visits (#)"
from sales.funnel
where visit_page_date between '2021-08-01' and '2021-08-31'
group by day_of_week
order by day_of_week 
```
After I finished my data analysis, I saved the query results in a CSV file and imported the data into Tableau and create the dashboard.<br/>

**Dashboard Breakdown:**
- The average ticket price represents the mean amount paid for each car.
- Lead conversion rate quantifies the percentage of leads (potential customers) successfully transformed into paying customers.
- Bar charts for a visual on the top-performing sales stores, brands, and states.
- Map for a visual on the top-performing sales states.
- Line and clustered column chart for visual on evolution of Revenue and Average Ticket Price of vehicles bough by month on the website.
- Line and clustered column chart for visual on number of Leads and Conversion Rates by month.
- Clustered Column Chart for visual on Days of the Week with the highest number of visits.<br/>

**Here are a few of my takeaways:**
- There isn't a strong correlation between the average ticket price and revenue growth, as average ticket price has remained relatively stable over the time period showed on the dashboard.
- There is a direct correlation between the leads conversion rate and the number of leads. When the conversion rate increases, the number of leads also rises, subsequently boosting revenue during the displayed time period.
- The day of the week with the highest number of visits is Monday.
- The top selling state is SP.
- The top selling brand is Fiat.
- The top selling store is Speedy Wheels Auto.<br/>	

Below you will find a picture of the final dashboard, but **if you want to access the live dashboard in Tableau public, please click on the following link:** https://public.tableau.com/app/profile/fabiana.fernandes/viz/Dash_vendas_16905561954870/Dashboard1

![alt text](https://github.com/fpfernandes/SQL_Tableau/blob/61c23545eb4529f2e7d7b48d3453a04e6a97de79/dash_tableau.PNG)

