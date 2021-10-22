# Relational Databases

Tag: SQL

**Constraints**

**Keys**

**Referential integrity** 

# Chapter 1

## 1. Query information_schema with SELECT

`information_schema` is a meta-database that holds information about your current database. `information_schema` has **multiple tables** you can query with the known `SELECT * FROM` syntax:

- `tables`: information about all *tables* in your current database
- `columns`: information about all *columns* in all of the tables in your current database

In this exercise, you'll only need information from the `'public'` schema, which is specified as the column `table_schema` of the `tables` and `columns` tables. The `'public'` schema holds information about user-defined tables and databases. The other types of `table_schema` hold system information – for this course, you're only interested in user-defined stuff.

```sql
-- Query the right table in information_schema
SELECT table_name 
FROM information_schema.tables
-- Specify the correct table_schema value
WHERE table_schema = 'public';

-- Query the right table in information_schema to get columns
SELECT column_name, data_type 
FROM information_schema.columns 
WHERE table_name = 'university_professors' AND table_schema = 'public';
```

## 2. CREATE your first few TABLEs

```sql
CREATE TABLE table_name (
 column_a data_type,
 column_b data_type,
 column_c data_type
);
```

```sql
ALTER TABLE table_name
ADD COLUMN column_name data_type
```

```sql
-- Create a table for the professors entity type
CREATE TABLE professors (
 firstname text,
 lastname text
);

-- Create a table for the universities entity type
Create Table universities(
    university_shortname text,
    university text,
    university_city text
);

-- Add the university_shortname column
ALTER TABLE professors
ADD COLUMN university_shortname text;
```

## 3. RENAME and DROP COLUMNs in affiliations

- To rename columns

```sql
ALTER TABLE table_name
RENAME COLUMN old_name TO new_name;
```

- To delete columns

```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

```sql
-- Rename the organisation column
ALTER TABLE affiliations
RENAME COLUMN organisation TO organization;
```

```sql
-- Delete the university_shortname column
ALTER TABLE affiliations
DROP COLUMN university_shortname;
```

## 4. Migrate data with INSERT INTO SELECT DISTINCT

```sql
INSERT INTO table_b. 
SELECT DISTINCT column_name1, column_name2, ... 
FROM table_a;
```

```sql
INSERT INTO table (column_name1, column_name2, column_name3) 
VALUES (value1, value2, value3);
```

```sql
DROP TABLE table_name;
```

```sql
-- Insert unique professors into the new table
INSERT INTO professors 
SELECT DISTINCT firstname, lastname, university_shortname 
FROM university_professors;

-- Insert unique affiliations into the new table
INSERT INTO affiliations 
SELECT DISTINCT firstname, lastname, function, organization 
FROM university_professors;
```

```sql
-- Delete the university_professors table
DROP TABLE university_professors;
```

# Chapter 2

## 1. Constraints

```sql
SELECT CAST(some_column AS integer)
FROM table;
```

```sql
-- Calculate the net amount as amount + fee
SELECT transaction_date, amount + CAST(fee AS integer) AS net_amount 
FROM transactions;
```

```sql
ALTER TABLE table_name
ALTER COLUMN column_name
TYPE varchar(10)
```

```sql
-- Specify the correct fixed-length character type
ALTER TABLE professors
ALTER COLUMN university_shortname
TYPE char(3);
```

### SUBSTRING

If you don't want to reserve too much space for a certain `varchar` column, you can *truncate* the values before converting its type.

For this, you can use the following syntax:

```sql
ALTER TABLE table_name
ALTER COLUMN column_name
TYPE varchar(x)
USING SUBSTRING(column_name FROM 1 FOR x)
```

```sql
-- Convert the values in firstname to a max. of 16 characters
ALTER TABLE professors 
ALTER COLUMN firstname 
TYPE varchar(16)
USING SUBSTRING(firstname FROM 1 for 16)
```

You should read it like this: Because you want to reserve only `x` characters for `column_name`, you have to retain a `SUBSTRING` of every value, i.e. the first `x` characters of it, and throw away the rest. This way, the values will fit the `varchar(x)` requirement.

### NOT-NULL and UNIQUE constraints

```sql
-- Disallow NULL values in firstname
ALTER TABLE professors 
ALTER COLUMN firstname SET NOT NULL;
```

Work for *new table*

```sql
CREATE TABLE table_name (
 column_name UNIQUE
);
```

add to an existing table

```sql
ALTER TABLE table_name
ADD CONSTRAINT some_name UNIQUE(column_name);
```

```sql
-- Make universities.university_shortname unique
ALTER TABLE universities
ADD CONSTRAINT university_shortname_unq UNIQUE(university_shortname);
```

## Chapter 3

![Untitled](Relational%20Databases/Untitled.png)

```sql
-- Count the number of distinct rows in universities
SELECT count(distinct(university_shortname, university, university_city)) 
FROM universities;
#output: 11

-- Count the number of distinct values in the university_city column
SELECT count(distinct(university_city)) 
FROM universities;
#output: 9
```

Great! So, obviously, the university_city column wouldn't lend itself as a key. Why? Because there are only 9 distinct values, but the table has 11 rows

### Primary keys

```sql
ALTER TABLE table_name
ADD CONSTRAINT some_name PRIMARY KEY (column_name)
```

```sql
-- Rename the organization column to id
ALTER TABLE organizations
Rename COLUMN organization TO id;

-- Make id a primary key
ALTER TABLE organizations
ADD CONSTRAINT organization_pk PRIMARY KEY (id);
```

### Surrogate keys

```sql
-- Add the new column to the table
ALTER TABLE professors 
ADD COLUMN id serial;

-- Make id a primary key
ALTER TABLE professors 
ADD CONSTRAINT professors_pkey PRIMARY KEY (id);

-- Have a look at the first 10 rows of professors
SELECT *
FROM professors
LIMIT 10;
```

**CONCATenate columns to a surrogate key**

```sql
-- Count the number of distinct rows with columns make, model
SELECT COUNT(Distinct(make, model))
FROM cars;

-- Add the id column
ALTER TABLE cars
ADD COLUMN id varchar(128);

-- Update id with make + model
UPDATE cars
SET id = CONCAT(make, model);

-- Make id a primary key
ALTER TABLE cars
ADD CONSTRAINT id_pk PRIMARY KEY(id);

-- Have a look at the table
SELECT * FROM cars;
```

```sql
-- Create the table
CREATE TABLE students (
  last_name varchar(128) not null,
  ssn int primary key,
  phone_no char(12) # char is for fixed length
);
```

## Chapter 4

### REFERENCE a table with a FOREIGN KEY

Table `a` should now refer to table `b`, via `b_id`, which points to `id`. `a_fkey` is, as usual, a constraint name you can choose on your own.

Pay attention to the **naming convention** employed here: Usually, a foreign key referencing another primary key with name `id` is named `x_id`, where `x` is the name of the referencing table in the singular form.

```sql
ALTER TABLE a 
ADD CONSTRAINT a_fkey FOREIGN KEY (b_id) REFERENCES b (id);
```

```sql
-- Add a foreign key on professors referencing universities
ALTER TABLE professors 
ADD CONSTRAINT professors_fkey FOREIGN KEY (university_id) REFERENCES universities (id);
```

As you can see, inserting a professor with non-existing university IDs violates the foreign key constraint you've just added. This also makes sure that all universities are spelled equally – adding to data consistency.

### JOIN tables linked by a foreign key

```sql
SELECT ...
FROM table_a
JOIN table_b
ON ...
WHERE ...
```

- `JOIN` `professors` with `universities` on `professors.university_id = universities.id`, i.e., retain all records where the foreign key of `professors` is equal to the primary key of `universities`.
- Filter for `university_city = 'Zurich'`.

```sql
-- Select all professors working for universities in the city of Zurich
SELECT professors.lastname, universities.id, universities.university_city
FROM professors
JOIN universities
ON professors.university_id = universities.id
WHERE universities.university_city = 'Zurich';
```

### How to implement N:M-relationships

![Untitled](Relational%20Databases%20/Untitled%201.png)

- create a table
- add foreign keys for every connected table
- add additional attributes
- no primary key!

```sql
CREATE TABLE affiliations(
	professor_id integer REFERENCES professor (id),
	organization_id varchar(256) REFERENCE organizations (id),
	function varchar(256)
);
```

Add a `professor_id` column with `integer` data type to `affiliations`, and declare it to be a foreign key that references the `id` column in `professors`.

```sql
ALTER TABLE table_name
ADD COLUMN column_name data_type REFERENCES other_table_name (other_column_name);
```

```sql
-- Add a professor_id column
ALTER TABLE affiliations
ADD COLUMN professor_id integer REFERENCES professors (id);
```

```sql
-- Rename the organization column to organization_id
ALTER TABLE affiliations
RENAME organization TO organization_id;

-- Add a foreign key on organization_id
ALTER TABLE affiliations
ADD CONSTRAINT affiliations_organization_fkey FOREIGN KEY (organization_id) REFERENCES organizations (id);
```

Update the `professor_id` column with the corresponding value of the `id` column in `professors`. 

"Corresponding" means rows in `professors` where the `firstname` and `lastname` are identical to the ones in `affiliations`

```sql
-- Set professor_id to professors.id where firstname, lastname correspond to rows in professors
UPDATE affiliations
SET professor_id = professors.id
FROM professors
WHERE affiliations.firstname = professors.firstname AND affiliations.lastname = professors.lastname;
```

### Referential integrity

A record in table A cannot point to a record in table B that does not exist. 

*A record referencing another table must refer to an existing record in that table*

You defined a foreign key on `professors.university_id` that references `university_id`, so referential integrity is said to hold from `professors` to `universities`.

This fails because referential integrity from `professors` to `university` is violated.

```sql
DELETE FROM universities WHERE id = 'EPF';
# this would raise an error

Error: update or delete on table "universities" violates foreign key constraint "professors_fkey" on table "professors"
DETAIL:  Key (id)=(EPF) is still referenced from table "professors".
```

### Change the referential integrity behavior of a key

Altering a key constraint: you have to **`DROP` the key constraint** and then **`ADD` a new one** with a different **`ON DELETE`** behavior. 

First, you need to know the name of the constraints, which is stored in `information_schema`

```sql
-- Identify the correct constraint name
SELECT constraint_name, table_name, constraint_type
FROM information_schema.table_constraints
WHERE constraint_type = 'FOREIGN KEY';
```

Second, delete the `affiliations_organization_id`_fkey foreign key constraint in `affiliations`.

```sql
-- Drop the right foreign key constraint
ALTER TABLE affiliations
DROP CONSTRAINT affiliations_organization_id_fkey;
```

Third Add a new foreign key to `affiliations` that `CASCADE`s deletion if a referenced record is deleted from `organizations`. Name it `affiliations_organization_id_fkey`.

```sql
-- Add a new foreign key constraint from affiliations to organizations which cascades deletion
ALTER TABLE affiliations
ADD CONSTRAINT affiliations_organization_id_fkey FOREIGN KEY (organization_id) REFERENCES organizations (id) ON DELETE CASCADE;
```

As you can see, whenever an organization referenced by an affiliation is deleted, the affiliations also gets deleted.

### Join all tables together

```sql
-- Filter the table and sort it
SELECT COUNT(*), organizations.organization_sector, 
professors.id, universities.university_city
FROM affiliations
JOIN professors
ON affiliations.professor_id = professors.id
JOIN organizations
ON affiliations.organization_id = organizations.id
JOIN universities
ON professors.university_id = universities.id
WHERE organizations.organization_sector = 'Media & communication'
GROUP BY organizations.organization_sector, 
professors.id, universities.university_city
ORDER BY COUNT DESC;
```

- you need to join all the tables,
- group by some column,
- and then use selection criteria to get only the rows in the correct sector.

Result: The professor with id 538 has the most affiliations in the "Media & communication" sector
