# SQL_Project-Analyzing-1M-Apple-Products-Sales-Records

## Exploratory Data Analysis (EDA)

## 1. Find the number of stores in each country.
```
select Country, count(store_id) as Total_Stores
from stores
group by country
order by 2 desc;
```

## 2. Calculate the total number of units sold by each store.
```
select Store_id, sum(Quantity) as Total_Units_Sold
from sales
group by store_id
order by 2 desc;
```

-- another way to solve it if we need the store's name as well (using joins)
```
select stores.store_id as Store_id, stores.store_name as Store_Name, sum(sales.quantity) as Total_Units_Sold 
from sales
left join stores
on sales.store_id = stores.store_id
group by 1,2
order by 3 desc;
```

## 3. Identify how many sales occurred in December 2023.
```
select count(sale_id) as Total_Sales_Count 
from sales
where sale_date >= '2023-12-01' AND sale_date <'2024-01-01';
```

## 4. Determine how many stores have never had a warranty claim filed.
```
select COUNT(*)
from stores
where store_id NOT IN (
						select distinct(store_id) 
						from sales AS s 
						right join warranty AS w 
						on s.sale_id = w.sale_id);
```

## 5. Calculate the percentage of warranty claims marked as "Warranty Void".
```
select Round((cast(count(claim_id) as decimal(10,2))/
		cast((select count(*) from warranty) as decimal(10,2)))*100,2)
		as Warranty_Void_Percent
from warranty 
where repair_status = 'Warranty Void';
```

## 6. Identify which store had the highest total units sold in the last year.
```
select extract(year from s.sale_date),st.store_name, s.store_id, Sum(s.quantity)
from sales as s 
join stores as st
on s.store_id = st.store_id
where extract(year from sale_date) = extract(year from current_date)-1  
group by 1,2,3
order by 4 desc
limit 1;
```

## 7. Count the number of unique products sold in the last year.
```
select count(distinct product_id)
from sales 
where extract(year from sale_date) = extract(year from current_date)-1;
```

## 8. Find the average price of products in each category.
```
select p.category_id,c.category_name, Round(avg(p.price),2) as Avg_Price
from products as p Join Category as c
on p.category_id = c.category_id
group by 1,2
order by 3 desc;
```

## 9. How many warranty claims were filed in 2020?
```
select count(claim_id) as Warranty_claim
from warranty
where claim_date >= '2020-01-01' AND claim_date < '2021-01-01';
```

## 10. For each store, identify the best-selling day based on the highest quantity sold.
```
With t1_cte as
(
select Store_id, to_char(sale_date, 'Day') as Day_name, Sum(Quantity) as Total_Units_Sold,
dense_rank() over(partition by store_id order by sum(quantity) desc) as Ranking
from sales
group by 1,2
) 
select * from t1_cte where ranking = 1;
```

## 11. Identify the least-selling product in each country for each year based on total units sold.
```
with product_rank_cte as
(
select st.country, s.product_id,p.product_name, extract(year from s.sale_date), sum(s.Quantity),
dense_rank() over(partition by st.country,extract(year from s.sale_date) order by sum(s.quantity) asc) as ranking
from Sales as s JOIN Stores as st
on s.store_id = st.store_id
join products as p
on s.product_id = p.product_id
group by 1,2,3,4
order by 1,4,5 asc
)
select * from product_rank_cte where ranking = 1;
```

## 12. warranty claims that were filed within 180 days of a product sale.
```
select 
	w.*,
	s.sale_date,
	w.claim_date-s.sale_date
from warranty as w Left JOIN sales as s
on s.sale_id = w.sale_id
where (w.claim_date-s.sale_date) between 0 and 180;
```

## 13. Determine how many warranty claims were filed for products launched in the last three years.
```
select p.product_name, count(w.claim_id) as no_claims, count(s.sale_id) as no_sales
from sales as s join products as p
on s.product_id = p.product_id left join warranty as w on s. sale_id = w.sale_id 
where launch_date >= current_date - interval '3 years'
Group by 1
Having count(w.claim_id) > 0; -- To ignore the products that have no claims
```

## 14. List the months in the last four years where sales exceeded 5,000 units in the USA.
```
select st. country, 
to_char(sale_date, 'MM-YYYY') AS month,
sum(s.quantity) as totals
from sales AS s JOIN stores AS st
on s.store_id = st.store_id
where st.country = 'USA' and s.sale_date >=current_date - interval '4 years'
group by 1,2
having sum(s.quantity)>5000
order by 1,2,3 asc;
```

## 15. Identify the product category with the most warranty claims filed in the last three years.
```
select c.category_name, count(claim_id) as Total_claims
from warranty as w
left join sales as s
on w.sale_id = s.sale_id
join products as p
on p.product_id = s.product_id
join category as c
on c.category_id = p.category_id
where w.claim_date >= current_date - interval '3 years'
group by 1
order by 2 desc;
```

## 16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```
select 
country, Total_units_sold, Total_claims,
Round(coalesce(Total_units_sold::numeric/nullif(Total_claims,0)::numeric * 100, 0),2)
from 
(
select st.country, sum(s.quantity) as Total_units_sold, count(w.claim_id) as Total_claims
from sales as s 
join stores as st
on st.store_id = s.store_id
left join warranty as w
on s.sale_id = w.sale_id
group by 1
order by 3 desc
) T1;
```

## 17. Analyze the year-by-year growth ratio for each store.
```
with yearly_sales as 
(
select s.store_id, st.store_name, extract(year from s.sale_date) as year, sum(p.price*s.quantity) as Total_sales
from sales as s
join products as p
on s.product_id = p.product_id
join stores as st
on s.store_id = st.store_id
group by 1,2,3
order by 1,2
),
Growth_ratio as 
(
select store_id, store_name, year, 
total_sales as current_year_sales,
lag(total_sales,1) over(partition by store_id,store_name order by year asc) as last_year_sales
from yearly_sales
)
select store_id, store_name, year,current_year_sales, last_year_sales, 
Round((current_year_sales - last_year_sales)/last_year_sales *100,2) as growth_percent
from growth_ratio
where last_year_sales IS NOT NULL AND year<> extract(year from current_date);
```

## 18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```
select distinct price from products order by 1 asc;
```
```
select
	case
		when p.price < 500 then 'Less Expensive Product'
		when p.price between 500 AND 1000 then 'Mid Range Product'
		else 'Expensive Product'
	end as price_segment,
	count(w.claim_id) as Total_claims
from warranty as w
left join sales as s
on w.sale_id = s.sale_id
join products as p
on p.product_id = s.product_id
where w.claim_date >= current_date - interval '6 years'
group by 1;
```
--Possible Correlation / Interpretation:
--Inverse correlation with price: The cheaper the product, the higher the number of claims.

## 19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```
with Total_claims_cte as 
(
select s.store_id, count(w.claim_id) as total_claims
from warranty as w
left join sales as s
on w.sale_id = s.sale_id
group by 1
),
Total_paid_repaired_cte as
(
select s.store_id, count(w.repair_status) as Total_Paid_Repaired
from warranty as w
left join sales as s
on w.sale_id = s.sale_id
where w.repair_status = 'Paid Repaired'
group by 1
)
select tc.store_id, st.store_name, tpr.total_paid_repaired, tc.total_claims,
Round((tpr.total_paid_repaired::numeric/tc.total_claims::numeric) * 100,2) as Percent_Paid_Repaired
from Total_claims_cte as tc
join Total_paid_repaired_cte as tpr
on tc.store_id = tpr.store_id
join stores as st
on tc.store_id = st.store_id
order by 5 desc
limit 1;
```

## 20. Write a query to calculate the monthly running total of sales for each store over the past five years and compare trends during this period.
```
with Monthly_Running_Total as 
(
select s.store_id, extract(year from s.sale_date) as year, to_char(s.sale_date, 'mm') as Month, sum(p.price * s.quantity) as Total_Revenue
from sales as s
join products as p
on s.product_id = p.product_id
where s.sale_date <= current_date - interval '5 years'
group by 1,2,3
order by 1,2,3 asc
)
select store_id,year,month,total_revenue,
sum(total_revenue) over (partition by store_id order by year,month asc)
from Monthly_Running_Total;
```

## 21. Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```
-- segmented into key periods
select s.sale_date, p.launch_date, s.quantity,
case when s.sale_date between p.launch_date and p.launch_date + interval '6 months' then '0-6 months'
	when s.sale_date between p.launch_date + interval '6 months' and p.launch_date + interval '12 months' then '6-12 months'
	when s.sale_date between p.launch_date + interval '12 months' and p.launch_date + interval '18 months' then '12-18 months'
	else '18+'
end as launch_segment
from sales as s
join products as p
on s.product_id = p.product_id;
```
-- analyzing product sales trends over segmented time period
```
select p.product_name,
case when s.sale_date between p.launch_date and p.launch_date + interval '6 month' then '0-6 months'
	when s.sale_date between p.launch_date + interval '6 month' and p.launch_date + interval '12 month' then '6-12 months'
	when s.sale_date between p.launch_date + interval '12 month' and p.launch_date + interval '18 month' then '12-18 months'
	else '18+'
end as launch_segment,
sum(s.quantity) as total_qty_sale
from sales as s
join products as p
on s.product_id = p.product_id
group by 1,2
order by 1,3 desc;
```


