# Database Design

Tag: SQL

# Chapter 1: Processing, Storing, and Organizing Data

**OLTP**: Online Transaction Processing

**OLAP**: Online Analytical Processing

![Untitled](Database%20Design/Untitled.png)

![Untitled](Database%20Design/Untitled%201.png)

![Untitled](Database%20Design/Untitled%202.png)

**Database models:** high-level specifications for database structure

- Most popular: relational model
- Some other options: NoSQL models, object-oriented model, network model

**Schemas:** blueprint of the database

- Defines tables, fields, relationships, indexes, and views
- When inserting data in relational databases, schemas must be respected

**Data Modeling**

1. Conceptual data model: describes entities, relationships, and attributes
2. Logical data model: defines tables, columns, relationships
3. Physical data model: describes physical storage

![Untitled](Database%20Design/Untitled%203.png)

![Untitled](Database%20Design/Untitled%204.png)

# Chapter 2: Database Schemas and Normalization

### Star schema (dimensional modeling)

![Untitled](Database%20Design/Untitled%205.png)

### Snowflake schemas

![Untitled](Database%20Design/Untitled%206.png)

**Normalizations**: identify repeating groups of data and create new tables for them; divides tables into smaller tables and connects them via relationships

**A more formal definition:**

> The goals of normalization are to:
> 
- Be able to characterize the level of redundancy in a relational schema
- Provide mechanisms for transforming schemas in order to remove redundancy

Find variables that are most likely to have repeating values

```sql
-- 1. create table with column
CREATE TABLE dim_author(
			author varchar(256) Not Null
);
-- 2. insert authors
INSERT INTO dim_author
SELECT distinct(author) from dim_book_star;

-- 3. add primary keys
Alter Table dim_author
ADD Column author_id SERIAL PRIMARY KEY;

-- 4. output the new table
Select * from dim_author;
```

## Normalized and Denormalized databases

disadvantages: complex queries require more CPU

Normalization saves space (do not need to store repeated strings)

![Untitled](Database%20Design/Untitled%207.png)

![Untitled](Database%20DesignUntitled%208.png)

**Normalization ensures better data integrity**

1. enforces data consistency
2. safe updating, removing, and inserting
3. easier to redesign by extending

![Untitled](Database%20Design/Untitled%209.png)

## Normal Forms

*Ordered from least to most normalized:*

### **First normal form (1NF)**

- Each record must be unique - no duplicate rows
- Each cell must hold one value

### **Second normal form (2NF)**

- Must satisfy 1NF AND
    - if primary key is one column
        - then automatically satisfies 2NF
    - if there is a composite primary key
        - then each non-key column must be dependent on all the keys

### **Third normal form (3NF)**

- Satisfies 2NF
- No transitive dependencies: non-key columns can't depend on other non-key columns

### Data anomalies

*What is risked if we don't normalize enough?*

1. update anomaly: data inconsistency caused by data redundancy when updating
2. insertion anomaly: unable to add a record due to missing attributes
3. deletion anomaly: deletion of records causes unintentional loss of data

# Chapter 3: Database Views

> In a database, a **view** is the result set of a stored query on the data, which the database users can query just as they would in a persistent database collection object
> 

Creating a view (syntax)

```sql
CREATE VIEW view_name AS
SELECT col1, col2
FROM table_name
WHERE condition;
```

![Untitled](Database%20Design/Untitled%2010.png)

Benefits:

- Doesn't take up storage
- A form of access control
- Masks complexity of queries

## Viewing views (in PostgreSQL)

```sql
SELECT * FROM INFORMATION_SCHEMA.views;
```

includes system views

```sql
SELECT * FROM information_schema.views
WHERE table_schema NOT IN ('pg_catalog', 'information_schema')
```

excludes system views

## Managing views

**Grating and revoking access to a view**

`GRANT privileges(s)` or `REVOKE privilege(s)`

- Privileges: `SELECT` `UPDATE`, etc

```sql
-- Revoke everyone's update and insert privileges
REVOKE update, insert on long_reviews FROM public; 

-- Grant the editor update and insert privileges 
GRANT update, insert on long_reviews TO editor;
```

**Update a view**

```sql
update films set kind = 'dramatic' where kind = 'drama';
```

`long_reviews` is updatable because it's made from one table and doesn't have any special clauses.

**Dropping a view**

```sql
drop view view_name [CASCADE | RESTRICT];
```

- `RESTRICT`(default): returns an error if there are objects that depend on the view
- `CASCADE`: drops view and any object that depends on that view

**Redefining a view**

```sql
CREATE OR REPLACE VIEW view_name AS new_query
```

- if a view with `view_name` exists, it is replaced
- `new_query` must generate the same column names, order, and data types as the old query
- the column output may be different
- new columns may be added at the end
- **if these criteria can't be met, drop the existing view and create a new one**

## Materialized views

- Stores the query results, not the query
- Querying a materialized view means accessing the stored query results
    - not running the query like a non-materialized view
- Refreshed or rematerialized when prompted or scheduled

When to use

- Long running queries; underlying query results don't change often; data warehouse because OLAP is not write-intensive; save on computational cost of frequent queries

```sql
CREATE MATERIALIZED VIEW my_mv AS SELECT * FROM existing_table;
REFRESH MATERIALIZED VIEW my_mv;
```

# Chapter 4: Database Management

Database roles

```sql
CREATE ROLE data_analyst;
CREATE ROLE intern with PASSWORD 'Passwordforintern' VALID UNTIL '2020-01-01';
CREATE ROLE admin CREATEDB; # create database
CREATE ROLE admin CREATEROLE; # create role

GRANT UPDATE ON ratings TO data_analyst;
REVOKE UPDATE ON ratings FROM data_analyst;
GRANT data_analyst TO alex;

ALTER role marta with PASSWORD 's3cur3p@ssw0rd';
```

table partitioning

![Untitled](Database%20Design/Untitled%2011.png)

![Untitled](Database%20Design/Untitled%2012.png)

![Untitled](Database%20Design/Untitled%2013.png)

Relation to sharding

Data integration

> Data integration combines data from different sources, formats, technologies to provide users with a translated and unified view of that data
> 

Picking a database management system (DBMS)

- flexible, reliable, scalable

![Untitled](Database%20Design/Untitled%2014.png)

![Untitled](Database%20Design/Untitled%2015.png)

![Untitled](Database%20Design/Untitled%2016.png)