# Apple-SQL-project
![Apple Logo](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/Apple_Changsha_RetailTeamMembers_09012021_big.jpg.slideshow-xlarge_2x.jpg) Apple Retail Sales 


## Project Overview

This project is designed to showcase advanced SQL querying techniques through the analysis of over 1 million rows of Apple retail sales data. The dataset includes information about products, stores, sales transactions, and warranty claims across various Apple retail locations globally. By tackling a variety of questions, from basic to complex, we'll demonstrate our ability to write sophisticated SQL queries that extract valuable insights from large datasets.

The project is ideal for data analysts looking to enhance their SQL skills by working with a large-scale dataset and solving real-world business questions.


## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.
```sql
create table stores(
	Store_ID varchar(10) primary key,
	Store_Name varchar(30),
	City varchar(25),
	Country varchar(25)
);
```

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.
```sql
Create table category(
	category_id	varchar(10) primary key,
	category_name varchar(20)
);
```

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.
```sql
create table products(
	product_id varchar(10) primary key,
	product_Name varchar(35),
	category_id varchar(10),
	launch_Date date,
	price float,
	constraint fk_category foreign key(category_id) references category(category_id)
);
```

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.
```sql
create table sales(
	sale_id varchar(15) primary key,
	sale_date date,
	store_id varchar(10),
	product_id varchar(10),
	quantity int,
	constraint fk_store foreign key(store_id) references stores(store_id),
	constraint fk_product foreign key(product_id) references products(product_id)
);
```

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).
```sql
create table warranty(
	claim_id varchar(10) primary key,
	claim_date date,
	sale_id varchar(15),
	repair_status varchar(15),
	constraint fk_sale foreign key(sale_id) references sales(sale_id)
);
```

## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

## Business Problems and Solutions

### Improving query performance
```sql
create index sales_product_id on sales(product_id);
create index sales_store_id on sales(store_id);
create index sales_sale_date on sales(sale_date);
```

### Q.1 Find the number of the stores in each country 
```sql
select 
	country,
	count(store_id) as total_store
from stores
group by 1
order by 2 desc;
```


 ### Q.2 Calculate the total number of units sold by each store
```sql
select 
	s.store_id,
	st.store_name,
	sum(s.quantity) as total_unit_sold
from sales as s
join stores as st
on s.store_id = st.store_id
group by 1, 2
order by 3 desc
;
```


### Q.3 Identify how many sales ocurred in December 2023 
```sql
select 
	count(*)
from sales
where 
	extract(year from sale_date) = 2023
	and
	extract(month from sale_date) = 12
;
```


### Q.4 Determine how many stores have never had warranty a claim filed
```sql
select count(*) from stores
where store_id not in
					(select 
						distinct store_id
					from sales as s
					right join warranty as w
					on s.sale_id = w.sale_id
					);
```


### Q.5 Calculate the percentage of warranty claims marked as 'Rejected'
```sql
select 
	round(count(claim_id)/(select count(claim_id) from warranty)::numeric * 100, 2) as total_warranty_rejected
from warranty
where repair_status = 'Rejected';
```


### Q.6 Identify which store had the highest total units sold in the last year
```sql
select 
	s.store_id,
	st.store_name,
	sum(s.quantity) as total_unit_sold
from sales as s
join stores as st
	on s.store_id = st.store_id
where sale_date >= current_date - interval '1 year'
group by 1, 2
order by 2 desc
limit 1;
```


 ### Q.7 Count the number of unique products sold in the last year
```sql
select 
	count(distinct product_id) as total_unique_product
from sales
where sale_date >= current_date - interval '1 year'
```


### Q.8 Find the average price of products in each category 
```sql
select 
	p.category_id,
	c.category_name,
	round(avg(price)::numeric, 2)
from products as p
join category as c
	on p.category_id = c.category_id
group by 1, 2;
```


### Q.9 How many warranty claims were submitted in 2020 and were repaired
```sql
select 
	count(*) 
from warranty
where 
	repair_status = 'Completed'
	and
	extract(year from claim_date) = 2024
;
```


 ### Q.10 For each store, identify the best selling day based on highest quantity sold
```sql
with day_table
as
(select
	store_id,
	to_char(sale_date, 'day') as day_name,
	sum(quantity) as total_sold,
	rank() over(partition by store_id order by sum(quantity) desc) as rank
from sales
group by 1, 2)
select 
	store_id,
	day_name,
	total_sold
from day_table
where rank = 1
order by 3 desc;
```


### Q.11 Identify at least selling product in each country for each year based on total units sold
```sql
with t1
as
(select 
	st.country,
	p.product_name,
	sum(s.quantity) as total_sold,
	rank() over(partition by st.country order by sum(s.quantity) desc) as rank
from sales as s
join stores as st
on s.store_id = st.store_id
join products as p
on s.product_id = p.product_id
group by 1, 2)
select * 
from t1
where rank = 1;
```


### Q.12 Calculate how many warranty claims were filed within 180 days of product sale
```sql
select 
	count(claim_id)
from warranty as w
left join sales as s
on w.sale_id = s.sale_id
where w.claim_date - s.sale_date <= 180
```


### Q.13 Determine how many warranty claims were filed for products launched in the last two year
```sql
select 
	p.product_name,
	count(w.claim_id) as no_claim
from warranty as w
join sales as s
	on w.sale_id = s.sale_id
join products as p
	on s.product_id = p.product_id
where p.launch_date >= current_date - interval '2 years'
group by 1; 
```


### Q.14 List the months in the last three years where sales exceeded 5.000 units in the USA 
```sql
select
	sum(s.quantity) as total_sale,
	to_char(sale_Date, 'MM-YYYY') as month_year
from sales as s
join stores as st
on s.store_id = st.store_id
where 
	s.sale_date >= current_date - interval '3 year'
	and
	st.country = 'United States'
	and 
	(select sum(quantity) from sales) >= 50000
group by 2
order by 1 desc;
```


 ### Q.15 Identify the product category with the most warranty claims filed in the last two year
```sql
select 
	c.category_name,
	count(claim_id) total_claim
from warranty as w
left join sales as s
	on w.sale_id = s.sale_id
join products as p
	on s.product_id = p.product_id
join category as c
	on c.category_id = p.category_id 
where claim_date >= current_date - interval '2 years'
group by 1
order by 2 desc;
```


 ### Q.16 Determine the percentage chance of receiving warranty claims after each purchase for each country
```sql
select 
	country,
	total_sold,
	total_claim,
	round(total_claim::numeric/total_sold::numeric * 100, 2) as risk
from 
(select 
	st.country,
	sum(s.quantity) as total_sold,
	count(w.claim_id) as total_claim
from sales as s
join stores as st
	on s.store_id = st.store_id
left join warranty as w
	on s.sale_id = w.sale_id
group by 1
order by 1 asc) 
order by 4 desc;
```


 ### Q.17 Analyze the year-by-year growth ratio for each store
```sql
with yearly_sales
as 
(select 
	st.store_id,
	st.store_name,
	extract(year from s.sale_date) as year,
	sum(quantity * p.price) as total_Sale
from sales as s
join stores as st
	on s.store_id = st.store_id
join products as p
	on p.product_id = s.product_id
group by 1, 2, 3
order by 1 asc),
growth_ratio
as 
(select 
	store_name,
	year,
	lag(total_sale, 1) over(partition by store_name order by year) as last_year_sale,
	total_sale as current_sale
from yearly_Sales)
select 
	store_name,
	year,
	last_year_sale,
	current_sale,
	round((current_sale-last_year_sale)::numeric/last_year_sale::numeric * 100, 2) as growth_ratio
from growth_ratio
where 
	last_year_sale is not null
	and
	year <> extract(year from current_date);
```


### Q.18 Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range
```sql
select 
	case
	 	when p.price < 500 then 'Less expenses product'
		when p.price between 500 and 1000 then 'Mid range product'
		when p.price > 500 then 'Expensive product'
		end as price_segment,
	count(w.claim_id)
from sales as s
join products as p
	on s.product_id = p.product_id
right join warranty as w
	on s.sale_id = w.sale_id
where s.sale_date >= current_date - interval '5 years'
group by 1;
```


### Q.19 Identify the stores with the highest percentage of "Completed" claims realative to total claims filed
```sql
with t1
as
(select 
	s.store_id,
	count(w.claim_id) as total_completed_claim
from warranty as w
join sales as s
	on w.sale_id = s.sale_id
where w.repair_status = 'Completed'
group by 1),
t2
as
(select 
	s.store_id,
	st.store_name,
	count(w.claim_id) as total_claim
from warranty as w
join sales as s
	on w.sale_id = s.sale_id
join stores as st
	on st.store_id = s.store_id
group by 1, 2)
select 
	t1.store_id,
	t2.store_name,
	round((t1.total_completed_claim::numeric/t2.total_claim::numeric)::numeric * 100, 2) as percentage_completed_claim,
	rank() over(order by round((t1.total_completed_claim::numeric/t2.total_claim::numeric)::numeric * 100, 2) desc)
from t1
join t2
	on t1.store_id = t2.store_id;
```


### Q.20 Write a querry to calculate the monthly running total of sales for each store over the past four years and compare trends during this point
```sql
with monthly_sales
as
(select 
	st.store_id,
	extract(year from s.sale_date) as year,
	extract(month from s.sale_date) as month,
	sum(s.quantity * p.price) as total_revenue
from sales as s
join stores as st
	on s.store_id = st.store_id
join products as p
	on p.product_id = s.product_id 
where s.sale_date >= current_date - interval '4 years'
group by 1, 2, 3
order by 1, 2, 3)
select 
	store_id,
	year,
	month,
	total_revenue,
	sum(total_revenue) over (partition by year, month order by store_id) as running_total
from monthly_sales;
```


### Q.21 Analyze product sales trends over time, segmented into key periods: from launch 6-12 month, 12-18 month, and beyond 18 months
```sql
select
	p.product_name,
	case 
		when s.sale_Date between p.launch_date and p.launch_Date + interval '6 month' then '0-6 month'
		when s.sale_Date between p.launch_date + interval '12 month' and p.launch_Date + interval '12 months' then '6-12 month'
		when s.sale_Date between p.launch_date + interval '12 month' and p.launch_Date + interval '18 months' then '6-18 month'
		else '18+ month'
	end as plc,
	sum(s.quantity) as total_sales_quantity
from sales as s
join products as p
	on s.product_id = p.product_id
group by 1, 2
order by 1, 3 desc;
```


## Conclusion

By completing this project, we can develop advanced SQL querying skills, improve your ability to handle large datasets, and gain practical experience in solving complex data analysis problems that are crucial for business decision-making. This project is an excellent addition to your portfolio and will demonstrate your expertise in SQL to potential employers.

---
