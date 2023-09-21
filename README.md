# Atliq hardware data exploration

#ERD(Entity relationship diagram) 
![image](https://github.com/bala942/atliq_hardware/assets/127521506/6ec0ba34-05fd-4ac0-80f4-25119ed950ee)

1.top market by net sales

```sql
SELECT market,
ROUND(
SUM(
((gp.gross_price*(1-pre.pre_invoice_discount_pct))*(1-(pro.discounts_pct+pro.other_deductions_pct)))
*mon.sold_quantity),0) 
AS total_gross_price
FROM 
	fact_gross_price gp INNER JOIN 
	fact_sales_monthly mon ON gp.product_code=mon.product_code 
	AND fiscial_year_converstion(date)=gp.fiscal_year INNER JOIN 
    fact_post_invoice_deductions pro ON mon.customer_code=pro.customer_code AND
    mon.product_code=pro.product_code AND mon.date=pro.date INNER JOIN 
	fact_pre_invoice_deductions pre ON pro.customer_code=pre.customer_code AND 
    fiscial_year_converstion(pro.date)=pre.fiscal_year INNER JOIN  
	dim_customer cus ON pre.customer_code=cus.customer_code 
WHERE gp.fiscal_year=2018 AND pre.fiscal_year=2018    
GROUP BY market 
ORDER BY total_gross_price desc
```
![image](https://github.com/bala942/atliq_hardware/assets/127521506/6c821135-731e-47f5-8a28-c44d14d20c19)

2.supply chain analytics using CTE
* net error
* absolute error
* forecast accuracy 

```sql
WITH total_qnt AS (
SELECT 
	c.customer_code,c.customer,market,
	SUM(fm.sold_quantity) AS total_sold_quantity,
	SUM(fm.forecast_quantity) AS total_forecast_quantity,
    SUM(forecast_quantity-sold_quantity) AS net_error,
    ROUND(SUM(forecast_quantity-sold_quantity)*100/SUM(forecast_quantity),1) 
		AS neterror_percentage,
	SUM(ABS((forecast_quantity-sold_quantity))) AS absolute_error,
    ROUND(SUM(ABS(forecast_quantity-sold_quantity))*100/SUM(forecast_quantity),1) 
		AS absolute_error_percentage
FROM 
	fact_forecast_merge_sales fm INNER JOIN 
	dim_customer c USING(customer_code)
WHERE fiscal_year=2021 
GROUP BY customer_code,customer,market)
SELECT 
	*,
    100-ABS(neterror_percentage) AS forecast_accuracy_neterror, 
    IF(absolute_error_percentage > 100,0,100-absolute_error_percentage) AS forecast_accuracy_absolute
FROM
	total_qnt
ORDER BY forecast_accuracy_neterror DESC;
```
![image](https://github.com/bala942/atliq_hardware/assets/127521506/f83c9887-714a-40ad-9dc9-0c9565e20dfa)

3.Top markets in specific region by gross sales(using window function)

```sql
WITH 
gross_total_market_region AS
(
SELECT market,region,ROUND(SUM(gross_price_total)/1000000,1) AS gross_sales_mln FROM gross_sales
where fiscal_year=2021
GROUP BY market,region),
rank_1 AS 
(
SELECT *,ROW_NUMBER() OVER(PARTITION BY region ORDER BY gross_sales_mln desc) AS rank_
FROM gross_total_market_region)
SELECT * FROM rank_1
WHERE rank_ IN(1,2,3)
```

![image](https://github.com/bala942/atliq_hardware/assets/127521506/1bf0123f-0978-4bd2-9574-f625fb33222a)

#power bi dashboard for indian market

![Screenshot 2023-09-21 231557](https://github.com/bala942/atliq_hardware/assets/127521506/7a0f09ab-4968-4adb-9204-0a11abdb175d)









