# Airline Company SQL Project

## INSIGHT TASKS
The purpose of these tasks is for you to work with and think about the type of data that you would use in the role and draw and explain the conclusions and recommendations you make to management. 

### TASK 1 - SQL

We have provided a simplified version of the data schema we use in the attachment. It is a star schema, with revenue transactions in the centre and descriptive dimensions linking to this.
Based on this schema please can you write some pseudocode (ideally SQL) to create summarised datasets that answer the following questions:

1. How many passengers are due to fly in the each of the next 3 months?
2. What is the average number of days between booking and flying split by booking country (based on billing address)?
3. Generate a list of mailable customer ids who have flown in the last 2 years, but don’t have any upcoming flights booked.

*Hint: All of these questions are based on flights, so you need to apply a filter based on ServiceCategory = ‘Flight’ (from the dimService table)*

#### Question 1: How many passengers are due to fly in the each of the next 3 months?
```sql
SELECT t3.Month as month_jurney,
	   t3.Year as year_jurney,
	   SUM(t1.TotalFeeUnits) as customers
	
FROM FactRevenues t1

JOIN DimDate t3 on (t1.DepartureDateSK = t3.DateSK)
JOIN DimService t4 on (t1.ServiceSK = t4.ServiceSK)

WHERE t4.ServiceCategory = 'Flight' 
AND t3.Month in ("October" "November" "December")
AND t3.Year = 2023

GROUP BY 1,2

-- NOTE: here passengers are counted round-trip
-- Another way to calculate passengers is written below

-- Step #1:
CREATE TABLE tabella_1 AS 
SELECT t1.BookingSK, 
	   t3.Month as month_jurney,
	   t3.Year as year_jurney,
	   MAX(t1.TotalFeeUnits) as numero_passeggeri,
	
FROM FactRevenues t1

JOIN DimDate t3 on (t1.DepartureDateSK = t3.DateSK)
JOIN DimService t4 on (t1.ServiceSK = t4.ServiceSK)

WHERE t4.ServiceCategory = 'Flight' 
AND t3.Month in ("October" "November" "December")
AND t3.Year = 2023

GROUP BY 1,2,3

-- Step #2:
CREATE TABLE tabella_2 as
SELECT  month_jurney,
		year_jurney,
		sum(numero_passeggeri) as totale_customer_per_months
		
FROM tabella_1

GROUP BY 1,2		

-- NOTE: here passengers are counted distinct
```

#### Question 2: What is the average number of days between booking and flying split by booking country (based on billing address)?
```sql
-- Step #1:
CREATE TABLE Fligths_Date as -- this is a support table

SELECT t1.BookingSK,
	   t2.BookingCountry,
	   CONVERT(t1.BookingDateSK, getdate(), 3) as booking_date,
	   CONVERT(t1.DepartureDateSK, getdate(), 3) as departure_date,
	   (calculated departure_date - calculated booking_date) as days_btw
	   	   
FROM FactRevenues t1

JOIN DimBooking t2 on (t1.BookingSK = t2.BookingSK)
JOIN DimService t3 on (t1.ServiceSK = t3.ServiceSK)

WHERE t3.ServiceCategory = 'Flight' 

-- Step #2:
CREATE TABLE Average_Days as 

SELECT BookingCountry,
	   AVG(days_btw) as average_days

FROM Fligths_Date

GROUP BY 1
```

#### Question 3: Generate a list of mailable customer ids who have flown in the last 2 years, but don’t have any upcoming flights booked.
```slq
-- Step #1:
CREATE TABLE booking_list as

SELECT DISTINCT t1.CustomerID,

FROM FactRevenues t1

JOIN DimDate t2 on (t1.BookingDateSK = t2.DateSK)
JOIN DimService t3 on (t1.ServiceSK = t3.ServiceSK)

WHERE  t3.ServiceCategory = 'Flight' 
AND t2.Date > today() 

-- Step #2:

CREATE TABLE mailable_customers as

SELECT DISTINCT t1.CustomerID,

FROM FactRevenues t1

JOIN DimDate t2 on (t1.DepartureDateSK = t2.DateSK)
JOIN DimService t3 on (t1.ServiceSK = t3.ServiceSK)

WHERE  t3.ServiceCategory = 'Flight' 
AND t2.Date BETWEEN todate("01/01/2021", "dd/mm/yyyy") AND todate("27/09/2023", "dd/mm/yyyy") 
AND t1.CustomerID NOT IN (Select * from booking_list)
```
