# Manipulating Data

Tag: SQL

# Chapter 1 Data Type

This is a good cheatsheet for data type: [https://tableplus.com/blog/2018/08/ms-sql-server-data-types-cheatsheet.html](https://tableplus.com/blog/2018/08/ms-sql-server-data-types-cheatsheet.html) 

## 1. Implicit Conversion

![Untitled](Manipulating%20Data/Untitled.png)

![Untitled](Manipulating%20Data/Untitled%201.png)

e.g. When comparing **decimals** to **integers**, the integer value is automatically converted to a decimal. Otherwise, the data after the decimal point would get lost.

## 2. Explicit Conversion

### CAST() Convert()

```sql
CAST(expression AS data_type [(length)])
CONVERT(data_type [(length)], expression[, style])
```

```sql
SELECT 
	-- Transform the year part from the birthdate to a string
	first_name + ' ' + last_name + ' was born in ' + CAST(YEAR(birthdate) AS nvarchar) + '.' 
FROM voters;

-- Output : Carol Rai was born in 1989

SELECT 
	total_votes
FROM voters
-- Transform the total_votes to char of length 10
WHERE CAST(total_votes AS char(10)) LIKE '5%';
```

```sql
SELECT 
	company, bean_origin, rating
FROM ratings
-- Convert the rating to an integer before comparison
WHERE CONVERT(int,rating) = 3;

SELECT
	first_name,last_name,
	-- Convert birthdate to varchar(10) to show it as yy/mm/dd
	CONVERT(varchar(10), birthdate, 11) AS birthdate,
    gender, country,
    -- Convert the total_votes number to nvarchar
    'Voted ' + CAST(total_votes AS nvarchar) + ' times.' AS comments
FROM voters
WHERE country = 'Belgium'
    -- Select only the female voters
	AND gender = 'F'
    -- Select only people who voted more than 20 times
    AND total_votes > 20;
```

# Chapter 2 Time

Most of the time, you can just use `GETDATE()` for retrieving the system's date and time

## 1. Extract Date Components

### YEAR(), MONTH(), DAY()

```sql
SELECT 
	first_name, last_name,
   	-- Extract the year of the first vote
	YEAR(first_vote_date)  AS first_vote_year,
    -- Extract the month of the first vote
	MONTH(first_vote_date) AS first_vote_month,
    -- Extract the day of the first vote
	DAY(first_vote_date)   AS first_vote_day
FROM voters
-- The year of the first vote should be greater than 2015
WHERE YEAR(first_vote_date) > 2015
-- The day should not be the first day of the month
  AND Day(first_vote_date) <> 1;
```

### DATENAME(datepart, date), DATEPART(datepart, date), DATEFROMPARTS(year, month, day)

`DATENAME()` and `DATEPART()` are two similar functions. The difference between them is that while the former understandably shows some date parts, as **strings of characters**, the latter returns only **integer** values.

![Untitled](Manipulating%20Data/Untitled%202.png)

```sql
SELECT 
	first_name, last_name, first_vote_date,
    -- Select the name of the month of the first vote
	DATENAME(mm, first_vote_date) AS first_vote_month
		-- Select the number of the day within the year
	DATENAME(dy, first_vote_date) AS first_vote_dayofyear
    -- Select day of the week from the first vote date
	DATENAME(weekday,first_vote_date) AS first_vote_dayofweek
FROM voters;
```

## 2. Arithmetic Operations on Dates

### DATEADD(), DATEDIFF()

- Subtract `@date1` from `@date2`.
- Add `@date1` to `@date2`.
- Using `DATEDIFF()`, calculate the difference in **years** between the results of the subtraction and the addition above.

```sql
DECLARE @date1 datetime = '2018-12-01';
DECLARE @date2 datetime = '2030-03-03';

SELECT
datediff(YEAR, @date2 - @date1, @date1+@date2)
```

```sql
SELECT 
	first_name,
	birthdate,
    -- Add 18 years to the birthdate
	DATEADD(Year, 18, birthdate) AS eighteenth_birthday
  FROM voters;
```

## 3. Validate date data type

### ISDATE()

### SET DATEFORMAT {format}

valid formats: `mdy`, `dmy`, `ymd`, `ydm`, `myd`, `dym`

```sql
DECLARE @date1 NVARCHAR(20) = '15/2019/4';

-- Set the date format and check if the variable is a date
SET DATEFORMAT dym;
SELECT ISDATE(@date1) AS result;
```

### SET LANGUAGE {language}

```sql
DECLARE @date1 NVARCHAR(20) = '12/18/55';

-- Set the correct language
SET LANGUAGE English;
SELECT
	@date1 AS initial_date,
    -- Check that the date is valid
	ISDATE(@date1) AS is_valid,
    -- Select the week day name
	DATENAME(weekday, @date1) AS week_day,
	-- Extract the year from the date
	YEAR(@date1) AS year_name;
```

# Chapter 3 Strings

## 1. Functions for positions

### Len()

```sql
SELECT TOP 10 
	company, 
	broad_bean_origin,
	-- Calculate the length of the broad_bean_origin column
	len(broad_bean_origin) AS length
FROM ratings
--Order the results based on the new column, descending
ORDER BY len(broad_bean_origin) DESC;
```

### CHARINDEX()

```sql
SELECT 
	first_name, last_name, email 
FROM voters
-- Look for the "dan" expression in the first_name
WHERE CHARINDEX('dan', first_name) > 0 
    -- Look for last_names that do not contain the letter "z"
	AND CHARINDEX('z', last_name) = 0;
```

### PATINDEX()

[Pattern](https://www.notion.so/aecdf9dafe07488cb73414aa82f392b0)

```sql
SELECT 
	first_name, last_name, email 
FROM voters
-- Look for first names that contain "rr" in the middle
WHERE PATINDEX('%rr%', first_name) > 0;
-- Look for first names that start with C and the 3rd letter is r
WHERE PATINDEX('C_r%', first_name) > 0;
-- Look for first names that have an "a" followed by 0 or more letters and then have a "w"
WHERE PATINDEX('%a%w%', first_name) > 0;
-- Look for first names that contain one of the letters: "x", "w", "q"
WHERE PATINDEX('%[xwq]%', first_name) > 0;
```

## 2. String transformation

### LOWER() UPPER()

### LEFT() RIGHT()

```sql
SELECT 
	first_name, last_name, country,
    -- Select only the first 3 characters from the first name
	LEFT(first_name, 3) AS part1,
    -- Select only the last 3 characters from the last name
    RIGHT(last_name, 3) AS part2,
    -- Select only the last 2 digits from the birth date
    RIGHT(birthdate, 2) AS part3,
    -- Create the alias for each voter
    LEFT(first_name, 3) + RIGHT(last_name, 3) + '_' + RIGHT(birthdate, 2) 
FROM voters;
```

### LTRIM() RTRIM() TRIM()

### REPLACE()

```sql
REPLACE(character_expression, searched_expression, replacement_expression)

SELECT 
	company AS old_company,
    -- Remove the text '(Valrhona)' from the name
	REPLACE(company, '(Valrhona)', '') AS new_company,
	bean_type,
	broad_bean_origin
FROM ratings
WHERE company = 'La Maison du Chocolat (Valrhona)';
```

### SUBSTRING()

```sql
SUBSTRING(character_expression, start, number_of_characters)
DECLARE @sentence NVARCHAR(200) = 'Apples are neither oranges nor potatoes.'
SELECT
	-- Extract the word "Apples" 
	SUBSTRING(@sentence, 1, 6) AS fruit1,
    -- Extract the word "oranges"
	SUBSTRING(@sentence, 20, 7) AS fruit2;
```

## 3. Manipulating groups of strings

### CONCAT() CONCAT_WS()

```sql
CONCAT(string1, string2 [, stringN ])
CONCAT_WS(separator, string1, string2 [, stringN ])

DECLARE @string1 NVARCHAR(100) = 'Chocolate with beans from';
DECLARE @string2 NVARCHAR(100) = 'has a cocoa percentage of';

SELECT 
	bean_type,
	bean_origin,
	cocoa_percent,
	-- Create a message by concatenating values with "+"
	@string1 + ' ' + bean_origin + ' ' + @string2 + ' ' + CAST(cocoa_percent AS nvarchar) AS message1,
	-- Create a message by concatenating values with "CONCAT()"
	CONCAT(@string1, ' ', bean_origin, ' ', @string2, ' ', cocoa_percent) AS message2,
	-- Create a message by concatenating values with "CONCAT_WS()"
	CONCAT_WS(' ', @string1, bean_origin, @string2, cocoa_percent) AS message3
FROM ratings
WHERE 
	company = 'Ambrosia' 
	AND bean_type <> 'Unknown';
```

### STRING_AGG() with group by

```sql
STRING_AGG(expression, separator) [WITHIN GROUP (ORDER BY expression)]
```

```sql
SELECT 
	company,
    -- Create a list with all bean origins ordered alphabetically
	STRING_AGG(bean_origin, ',') WITHIN GROUP (ORDER BY bean_origin) AS bean_origins
FROM ratings
WHERE company IN ('Bar Au Chocolat', 'Chocolate Con Amor', 'East Van Roasters')
-- Specify the columns used for grouping your data
GROUP BY company;
```

### STRING_SPLIT()

```sql
STRING_SPLIT(string, separator)
```

```sql
DECLARE @phrase NVARCHAR(MAX) = 'In the morning I brush my teeth. In the afternoon I take a nap. In the evening I watch TV.'

SELECT value
FROM STRING_SPLIT(@phrase, '.');
```

```sql
SELECT
    -- Concatenate the first and last name
	CONCAT('***' , first_name, ' ', UPPER(last_name), '***') AS name,
    -- Mask the last two digits of the year
    REPLACE(birthdate, SUBSTRING(CAST(birthdate AS varchar), 3, 2), 'XX') AS birthdate,
	email,
	country
FROM voters
   -- Select only voters with a first name less than 5 characters
WHERE LEN(first_name) < 5
   -- Look for this pattern in the email address: "j%[0-9]@yahoo.com"
	AND PATINDEX('j_a%@yahoo.com', email) > 0;
```

# Chapter 4 Numeric Data Properties

## 1. Aggregate

### COUNT()

```sql
COUNT([ALL] expression)
COUNT(DISTINCT expression)
COUNT(*)

SELECT 
	gender, 
	-- Count the number of voters for each group
	count(*) AS voters,
	-- Calculate the total number of votes per group
	sum(total_votes) AS total_votes
FROM voters
GROUP BY gender;
```

## 2. Analytic

### FIRST_VALUE() LAST_VALUE()

- Returns the first/last value in an ordered set

```sql
FIRST_VALUE(numeric_expression) OVER ([PARTITION BY column] ORDER BY column ROW_or_RANGE frame)

LAST_VALUE(numeric_expression) OVER ([PARTITION BY column] ORDER BY column ROW_or_RANGE frame)

SELECT 
	first_name + ' ' + last_name AS name,
	country,
	birthdate,
	-- Retrieve the birthdate of the oldest voter per country
	FIRST_VALUE(birthdate) 
	OVER (PARTITION BY country ORDER BY birthdate) AS oldest_voter,
	-- Retrieve the birthdate of the youngest voter per country
	LAST_VALUE(birthdate) 
		OVER (PARTITION BY country ORDER BY birthdate ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
				) AS youngest_voter
FROM voters
WHERE country IN ('Spain', 'USA');
```

- Also, if you want to function to be applied on all the rows in your partition, you can add this command in the `OVER()` clause: `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`

### LAG() and LEAD()

- access data from previous/next row

```sql
SELECT 
	first_name,
	last_name,
	total_votes AS votes,
    -- Select the number of votes of the next voter
	LEAD(total_votes) OVER (order by total_votes) AS votes_next_voter,
    -- Calculate the difference between the number of votes
	LEAD(total_votes) Over (ORDER BY total_votes) - total_votes AS votes_diff
FROM voters
WHERE country = 'France'
ORDER BY total_votes;

SELECT 
	broad_bean_origin AS bean_origin,
	rating,
	cocoa_percent,
    -- Retrieve the cocoa % of the bar with the previous rating
	LAG(cocoa_percent) 
		OVER(PARTITION BY broad_bean_origin ORDER BY rating) AS percent_lower_rating
FROM ratings
WHERE company = 'Fruition'
ORDER BY broad_bean_origin, rating ASC;
```

## 3. Mathematical

### ABS() SIGN()

- `SIGN()` is an indication of whether the number is positive (+1) or negative (-1).

### CEILING() FLOOR() ROUND( , length)

- smallest integer greater than or equal to the expression
- largest integer less than or equal to the expression
- numeric value, rounded to the specific length

```sql
SELECT
	rating,
	-- Round-up the rating to an integer value
	CEILING(rating) AS round_up,
	-- Round-down the rating to an integer value
	FLOOR(rating) AS round_down,
	-- Round the rating value to one decimal
	ROUND(rating, 1) AS round_onedec,
	-- Round the rating value to two decimals
	ROUND(rating, 2) AS round_twodec
FROM ratings;
```

### POWER(), SQUARE(), SQRT()

```sql
DECLARE @number DECIMAL(4, 2) = 4.5;
DECLARE @power INT = 4;

SELECT
	@number AS number,
	@power AS power,
	-- Raise the @number to the @power
	POWER(@number, @power) AS number_to_power,
	-- Calculate the square of the @number
	SQUARE(@number) num_squared,
	-- Calculate the square root of the @number
	SQRT(@number) num_square_root;
```