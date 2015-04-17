
Improve Queries Performance with PostgreSQL
===================

To improve query performance you must know the following topics:
-----------------------------------------------------------------
 1. Types of indexes in PostgreSQL and their use cases.
 2. The execution plan for each of your queries using explain and analyze.
 3. How to work with subsets of information before joining tables.
 4. Retrieving only the information you need from your tuples.


Indexes in PostgreSQL
-------------

#### B-Tree index, is the most common type of index, it is used in the following cases:

- Primary keys (each primary key is by default a unique b-tree index)
- Foreign keys (you must create a unique b-tree index for each foreign key)
- Improves performance on operations like >, <, >=, <=, =
- Searching with LIKE operator for strings that start with: LIKE pattern%    

```sql
CREATE INDEX index_name ON table_name column_name
DROP INDEX index_name
```

#### GIN index, specially aimed for FTS or searching inside composite values (jsonb, hstore...):
- Best for static data lookups
- Slower to create/update
- Supports operators on one-dimensional arrays
- Improves performance on operations like: ilike '%pattern%' and FTS with addons like trigram, unaccent...


Activate trgm extensión for fast searches with similarity or like, ilike operators.
 
 ```sql
 CREATE EXTENSION pg_trgm;
 DROP EXTENSION pg_trgm;
    
 CREATE INDEX index_name ON table_name USING  GIN(column_name gin_trgm_ops)

 Example:
 CREATE INDEX trgm_idx ON test_trgm USING gin (t gin_trgm_ops);
 SELECT * FROM test_trgm WHERE t LIKE '%foo%bar';
 ```

#### GIST index, specially aimed for FTS and Two-dimension geometrical data types:
- Best for dynamic data lookups
- Faster to create/update
- Supports operators on two-dimensional geometric data types and location
- Improves performance on operations like: ilike '%pattern%' and FTS with addons like trigram, unaccent...

```sql
    CREATE INDEX index_name ON table_name USING GIST(column_name gist_trgm_ops)
```

#### Partial indexes, specially aimed to create portions of a dataset:
- Useful in situations when you want to access a subset of your table under specific filters.
- Are marked by specific conditions where total > 1000, where state = 'complete'.

 ```sql
CREATE INDEX index_name ON table_name (column_name) WHERE CONDITION

Examples:

CREATE TABLE tests (
    subject text,
    target text,
    success boolean
);

CREATE UNIQUE INDEX tests_success_constraint ON tests (subject, target) WHERE success;

CREATE INDEX orders_unbilled_index ON orders (order_nr)
    WHERE billed is not true;

    
SELECT * FROM orders WHERE billed is not true AND order_nr < 10000;
```

#### Multi-column indexes, specially aimed for queries requiring 2 or more columns regularly:

- Useful in situations when you want to access a subset of your table according to the a condition applied to 2 colums.
- A query using event_id = 1 AND state='complete' will have the benefits of this kind of index

 ```sql
CREATE INDEX [index_name] ON [table_name] (column_name, column_name)

CREATE TABLE test2 (
  major int,
  minor int,
  name varchar
);

CREATE INDEX test2_mm_idx ON test2 (major, minor);

SELECT name FROM test2 WHERE major = constant AND minor = constant;
 ```
