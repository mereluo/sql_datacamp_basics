# Intermediate SQL Server

Tag: SQL

# Chapter 1 Summarizing Data

One of the first steps in data analysis is examining data through aggregations. This chapter explores how to create aggregations in SQL Server, a common first step in data exploration. You will also clean missing data and categorize data into bins with CASE statements

## 1. Summary Statistics

```sql
-- Calculate the aggregations by Shape
SELECT Shape,
       AVG(DurationSeconds) AS Average, 
       MIN(DurationSeconds) AS Minimum, 
       MAX(DurationSeconds) AS Maximum
FROM Incidents
GROUP BY Shape
-- Return records where minimum of DurationSeconds is greater than 1
HAVING MIN(DurationSeconds) > 1;
```

> What is the difference between `HAVING` and `WHERE`?
> 

The main difference between them is that **the WHERE clause is used to specify a condition for filtering records before any groupings are made**, while the HAVING clause is used to specify a condition for filtering values from a group.

## 2. Missing Values

```sql
SELECT *
FROM Incidents
WHERE Shape IS NOT NULL
```

```sql
-- Return the specified columns
SELECT IncidentDateTime, IncidentState
FROM Incidents
-- Exclude all the missing values from IncidentState  
WHERE IncidentState IS NOT NULL;
```

### ISNULL()

what if you want to replace the missing values with another value instead of omitting them? You can do this using the `ISNULL()` function. Here we replace all the missing values in the `Shape` column using the word `'Saucer'`

```sql
SELECT  Shape, ISNULL(Shape, 'Saucer') AS Shape2
FROM Incidents
```

You can also use `ISNULL()` to replace values from a different column instead of a specified word.

```sql
-- Check the IncidentState column for missing values and replace them with the City column
SELECT IncidentState, ISNULL(IncidentState, City) AS Location
FROM Incidents
-- Filter to only return missing values from IncidentState
WHERE IncidentState IS NULL;
```

### COALESCE

What if you want to replace missing values in one column with another and want to check the replacement column to make sure it doesn't have any missing values? To do that you need to use the `COALESCE` statement.

```sql
SELECT Shape, City, COALESCE(Shape, City, 'Unknown') as NewShape
FROM Incidents
+----------------+-----------+-------------+
| Shape          |  City     |  NewShape   |
+----------------+-----------+-------------+
| NULL           | Orb       | Orb         |
| Triangle       | Toledo    | Triangle    |
| NULL           | NULL      | Unknown     | 
+----------------+-----------+-------------+
```

Replace missing values in `Country` with the first non-missing value from `IncidentState` or `City`, in that order. Name the new column `Location`.

```sql
-- Replace missing values 
SELECT Country, COALESCE(Country, IncidentState, City) AS Location
FROM Incidents
WHERE Country IS NULL
```

## 3. Case Conditioning

- Create a new column, `SourceCountry`, defined from these cases:
    - When `Country` is `'us'` then it takes the value `'USA'`.
    - Otherwise it takes the value `'International'`.

```sql
SELECT Country, 
       CASE WHEN Country = 'us'  THEN 'USA'
       ELSE 'International'
       END AS SourceCountry
FROM Incidents
```

```sql
-- Complete the syntax for cutting the duration into different cases
SELECT DurationSeconds, 
-- Start with the 2 TSQL keywords, and after the condition a TSQL word and a value
      CASE WHEN (DurationSeconds <= 120) THEN 1
-- The pattern repeats with the same keyword and after the condition the same word and next value          
       WHEN (DurationSeconds > 120 AND DurationSeconds <= 600) THEN 2
-- Use the same syntax here             
       WHEN (DurationSeconds > 601 AND DurationSeconds <= 1200) THEN 3
-- Use the same syntax here               
       WHEN (DurationSeconds > 1201 AND DurationSeconds <= 5000) THEN 4
-- Specify a value      
       ELSE 5 
       END AS SecondGroup   
FROM Incidents
```

# Chapter 2 Math Functions

This chapter explores essential math operations such as rounding numbers, calculating squares and square roots, and counting records. You will also work with dates in this chapter!

## 1. Counting the total

Write a T-SQL query which will return the sum of the `Quantity` column as `Total` for each type of `MixDesc`.

```sql
-- Write a query that returns an aggregation 
SELECT MixDesc, SUM(Quantity) AS Total
FROM Shipments
-- Group by the relevant column
GROUP BY MixDesc;
```

you will calculate the number of orders for each concrete type. Since each row represents one order, all you need to is count the number of rows for each type of `MixDesc`.

```sql
-- Count the number of rows by MixDesc
SELECT MixDesc, Count(*)
FROM Shipments
GROUP BY MixDesc
```

## 2. Dates

### DATEDIFF

Write a query that returns the number of days between `OrderDate` and `ShipDate`.

```sql
-- Return the difference in OrderDate and ShipDate
SELECT OrderDate, ShipDate, 
       DATEDIFF(DD, OrderDate, ShipDate) AS Duration
FROM Shipments
```

### DATEADD

```sql
-- Return the DeliveryDate as 5 days after the ShipDate
SELECT OrderDate, 
       DATEADD(DD, 5, ShipDate) AS DeliveryDate
FROM Shipments
```

## 3. Rounding and Truncating

```sql
Round(number, length, [,function])
```

```sql
-- Round Cost to the nearest dollar
SELECT Cost, 
       ROUND(Cost, 0) AS RoundedCost
FROM Shipments
```

```sql
-- Truncate cost to whole number
SELECT Cost, 
       ROUND(Cost, 0, 1) AS TruncateCost
FROM Shipments
```

## 4. More Math

### ABS()

```sql
-- Return the absolute value of DeliveryWeight
SELECT DeliveryWeight,
       ABS(DeliveryWeight) AS AbsoluteValue
FROM Shipments
```

### SQRT() SQUARE()

```sql
-- Return the square and square root of WeightValue
SELECT WeightValue, 
       SQUARE(WeightValue) AS WeightSquare, 
       SQRT(WeightValue) AS WeightSqrt
FROM Shipments
```

### LOG(number, [,Base])

# Chapter 3 Processing Data

You will create variables and write while loops to process data. You will also write complex queries by using derived tables and common table expressions.

## 1. While loop

- Variables are needed to set values `DECLARE @variablename data_tye`
- Variables:
    - `VARCHAR(n)`, `INT`
    - `DECIMAL(p, s)` or `NUMERIC(p, s)`
        - `p`: total number of decimal digits that will be stored, both to the left and to the right of the decimal point
        - `s`: number of decimal digits that will be stored to the right of the decimal point

```sql
-- Declare ctr as an integer
DECLARE @ctr INT
-- Assign 1 to ctr
SET @ctr = 1
-- Specify the condition of the WHILE loop
WHILE @ctr < 10
-- Begin the code to execute inside WHILE loop
	BEGIN
-- Keep incrementing the value of @ctr
		SET @ctr = @ctr + 1
-- Check if ctr is equal to 4
    IF @ctr = 4
-- When ctr is equal to 4, the loop will break
       BREAK
-- End WHILE loop
END
```

## 2. Derived tables

You can use derived tables when you want to break down a complex query into smaller steps. 

A derived table is a query which is used in the place of a table. 

Derived tables are a great solution if you want to create intermediate calculations that need to be used in a larger query.

```sql
SELECT a.* FROM Kidney a
-- This derived table computes the Average age joined to the actual table
JOIN (SELECT AVG(age) AS AverageAge
			FROM kidney) b
ON a.Age = b.AverageAge
```

You will calculate the maximum value of the blood glucose level for each record by age.

```sql
SELECT a.RecordId, a.Age, a.BloodGlucoseRandom, 
-- Select maximum glucose value (use colname from derived table)
       b.MaxGlucose
FROM Kidney a
-- Join to derived table
JOIN (SELECT Age, MAX(BloodGlucoseRandom) AS MaxGlucose FROM Kidney GROUP BY Age) b
-- Join on Age
ON a.Age = b.Age
```

Create a derived table to return all patient records with the highest `BloodPressure` at their `Age` level.

```sql
SELECT *
FROM Kidney a
-- Create derived table: select age, max blood pressure from kidney grouped by age
JOIN (Select Age, MAX(BloodPressure) AS MaxBloodPressure FROM Kidney GROUP BY Age) b
-- JOIN on BloodPressure equal to MaxBloodPressure
ON a.BloodPressure = b.MaxBloodPressure
-- Join on Age
AND a.Age = b.Age
```

## 3. Common Table Expressions (CTE)

Also a type of temporary table

```sql
-- CTE definitions start with the keyword WITH
-- Followed by the CTE nams and the clolumns it contains
WITH CTEName (Col1, Col2)
AS
-- Define the CTE query
(
-- The two columns from the definitionabove
		 SELECT Col1, Col2
		 FROM TableName
)
```

You will use a CTE to return **all the ages** with the maximum `BloodGlucoseRandom` in the table.

- Create a CTE `BloodGlucoseRandom` that returns one column (`MaxGlucose`) which contains the maximum `BloodGlucoseRandom` in the table.
- Join the CTE to the main table (`Kidney`) on `BloodGlucoseRandom` and `MaxGlucose`.

```sql
-- Specify the keyowrds to create the CTE
WITH BloodGlucoseRandom (MaxGlucose) 
AS (SELECT MAX(BloodGlucoseRandom) AS MaxGlucose FROM Kidney)

SELECT a.Age, b.MaxGlucose
FROM Kidney a
-- Join the CTE on blood glucose equal to max blood glucose
JOIN BloodGlucoseRandom b
ON a.BloodGlucoseRandom = b.MaxGlucose
```

# Chapter 4 Window Functions

In the final chapter of this course, you will work with partitions of data and window functions to calculate several summary stats and see how easy it is to create running totals and compute the mode of numeric columns.

## 1. Window Functions with Aggregations

```sql
-- Create a Window data Grouping
OVER (PARTITION BY SalesYear ORDER BY SalesYear)
```

Write a T-SQL query that returns the sum of `OrderPrice` by creating partitions for each `TerritoryName`.

```sql
SELECT OrderID, TerritoryName, 
       -- Total price for each partition
       SUM(OrderPrice) 
       -- Create the window and partitions
       OVER(PARTITION BY TerritoryName) AS TotalPrice
FROM Orders
```

You will calculate the number of orders in each territory.

```sql
SELECT OrderID, TerritoryName, 
       -- Number of rows per partition
       COUNT(OrderID) 
       -- Create the window and partitions
       OVER(PARTITION BY TerritoryName) AS TotalOrders
FROM Orders
```

## 2. Common Window Functions

### FIRST_VALUE(), LAST_VALUE()

- First, create partitions for each territory
- Then, order by `OrderDate`
- Finally, use the `FIRST_VALUE()` and/or `LAST_VALUE()` functions as per your requirement

```sql
SELECT TerritoryName, OrderDate, 
       -- Select the first value in each partition
       FIRST_VALUE(OrderDate) 
       -- Create the partitions and arrange the rows
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS FirstOrder
FROM Orders
```

### LEAD()

- First, create partitions
- Then, order by a certain column
- Finally, use the `LEAD()` and/or `LAG()` functions as per your requirement

```sql
SELECT TerritoryName, OrderDate, 
       -- Specify the **previous** OrderDate in the window
       LAG(OrderDate) 
       -- Over the window, partition by territory & order by order date
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS PreviousOrder,
       -- Specify the **next** OrderDate in the window
       LEAD(OrderDate) 
       -- Create the partitions and arrange the rows
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS NextOrder
FROM Orders
```

## 3. More complex

### ROW_NUMBER()

```sql
SELECT TerritoryName, OrderDate, 
       -- Create a running total
       SUM(OrderPrice) 
       -- Create the partitions and arrange the rows
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS TerritoryTotal	  
FROM Orders
```

Write a T-SQL query that assigns row numbers to all records partitioned by `TerritoryName` and ordered by `OrderDate`.

```sql
SELECT TerritoryName, OrderDate, 
       -- Assign a row number
       ROW_NUMBER() 
       -- Create the partitions and arrange the rows
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS OrderCount
FROM Orders
```

## 4. Statistical Functions

### STDEV() Standard Deviation

Create the window, partition by `TerritoryName` and order by `OrderDate` to calculate a running standard deviation of `OrderPrice`.

```sql
SELECT OrderDate, TerritoryName, 
       -- Calculate the standard deviation
	   STDEV(OrderPrice) 
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS StdDevPrice	  
FROM Orders
```

### Mode

- First, create a CTE containing an ordered count of values using `ROW_NUMBER()`
- Write a query using the CTE to pick the value with the highest row number

```sql
-- Create a CTE Called ModePrice which contains two columns
WITH ModePrice (OrderPrice, UnitPriceFrequency)
AS
(
	SELECT OrderPrice, 
	ROW_NUMBER() 
	OVER(PARTITION BY OrderPrice ORDER BY OrderPrice) AS UnitPriceFrequency
	FROM Orders 
)

-- Select everything from the CTE
SELECT *
FROM ModePrice
```

All you need to do now is to find the `OrderPrice` with the highest row number.

Use the CTE `ModePrice` to return the value of `OrderPrice` with the highest row number.

```sql
-- Select the order price from the CTE
SELECT OrderPrice AS ModeOrderPrice
FROM ModePrice
-- Select the maximum UnitPriceFrequency from the CTE
WHERE UnitPriceFrequency IN (SELECT MAX(UnitPriceFrequency) FROM ModePrice)
```