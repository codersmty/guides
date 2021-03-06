
Improve Queries Performance with PostgreSQL
===================

####Improving query performance involves the following topics:
 1. Knowing the types of indexes in PG and their use cases.
 2. Use Explain and Analyze to dig into query execution plans.
 3. Aggreation and subsets of tables beforing applying joins.
 4. Retrieve only the columns you need from your tuples.


1. Knowing the types of indexes in PG and their use cases
-------------

#### B-Tree index, is the most common type of index, it is used in the following cases:

- Primary keys (each primary key is by default a unique b-tree index)
- Foreign keys (you must create a unique b-tree index for each foreign key)
- Improves performance on operations like >, <, >=, <=, =
- Searching with LIKE operator for strings that start with: LIKE pattern%    

```sql
# How to create a B-Tree index:
CREATE [UNIQUE] INDEX index_name ON table_name column_name
DROP INDEX index_name

# Examples that benefit form a B-Tree index:
# Searching by a primary key
SELECT * from users where user.id = 1

# Joining tables via a foreign key
SELECT * from users INNER JOIN addresses ON users.id = addresses.user_id

# Search pattern with ilike, like hello%
SELECT * from users where ilike 'arm%'

# Search with >, <, >=, = operators
SELECT * from orders where total > 1000
SELECT * from orders where status = 'complete''
```

#### GIN index, specially aimed for FTS or searching inside composite values (jsonb, hstore...):
- Best for static data lookups
- Slower to create/update
- Supports operators on one-dimensional arrays
- Improves performance on operations like: ilike '%pattern%' and FTS with addons like trigram, unaccent...

```sql
# On a non ts_vector you must activate trgm extension for fast searches with
# similarity (sinonyms) or like, ilike operators.
 
# How to activate/deactivate pg_trgm extension
CREATE EXTENSION pg_trgm;
DROP EXTENSION pg_trgm;
 
# How to create/drop a GIN index:   
CREATE INDEX index_name ON table_name USING  GIN(column_name gin_trgm_ops)
DROP index_name

# Examples that benefit from a GIN index:
SELECT * FROM users WHERE email ILIKE '%juan@g%'
```

#### GIST index, specially aimed for FTS and Two-dimension geometrical data types:
- Best for dynamic data lookups
- Faster to create/update
- Supports operators on two-dimensional geometric data types and location
- Improves performance on operations like: ilike '%pattern%' and FTS with addons like trigram, unaccent...

```sql
# On a non ts_vector you must activate trgm extension for fast searches with
# similarity (sinonyms) or like, ilike operators.
 
# How to activate/deactivate pg_trgm extension
CREATE EXTENSION pg_trgm;
DROP EXTENSION pg_trgm;
 
# How to create/drop a GIST index:   
CREATE INDEX index_name ON table_name USING  GIST(column_name gist_trgm_ops)
DROP index_name

# Examples that benefit from a GIST index:
SELECT * FROM users WHERE email ILIKE '%juan@g%'
```

#### Partial indexes, specially aimed to create portions of a dataset:
- Useful in situations when you want to access a subset of your table under specific filters.
- Are marked by specific conditions where total > 1000, where state = 'complete'.

```sql
 
# How to create/drop a Partial index:
CREATE INDEX index_name ON table_name (column_name) WHERE CONDITION
DROP index_name

# Unique composite index with 2 colums and 1 condition
CREATE UNIQUE INDEX tests_success_constraint ON tests (subject, target) WHERE success = true;

# Partial index where order_nr is not true
CREATE INDEX orders_unbilled_index ON orders (order_nr) WHERE billed is not true;

# Examples that benefit from a Partial index:
SELECT * FROM orders WHERE billed is not true AND order_nr < 10000;
SELECT * FROM tests WHERE subject = 1 AND target = 2 WHERE success = true;
```

#### Multi-column indexes, specially aimed for queries requiring 2 or more columns regularly:

- Useful in situations when you want to access a subset of your table according to the a condition applied to 2 colums.
- A query using order_id = 1 AND state='complete' will have the benefits of this kind of index

```sql
# How to create/drop a Multi Column index:
CREATE INDEX index_name ON table_name (column_one_name, column_two_name)
CREATE INDEX test2_mm_idx ON test2 (major, minor);
# Composite primary key
CREATE UNIQUE INDEX index_order_id_user_id ON order_summaries (order_id, column_id)

# Examples that benefit from a Multi Column index:
SELECT name FROM test2 WHERE major = 'math' AND minor = 'music';
```

2. Use Explain and Analyze to dig into query execution plans.
--------------------------------------------------------------

#### Real execution time query plans

- By default PostgreSQL uses the system cache to store the latest executed queries, in order to have a real executing time and metrics we must clear the cache before running a certain query plan:

```
# In Linux
pg_ctl stop
sudo su -
sync
echo 3 > /proc/sys/vm/drop_caches
logout
pg_ctl start -l $PGLOG

# In Mac OS X
sudo purge
```

#### EXPLAIN and ANALYZE basics

- By prepending the command EXPLAIN and ANALYZE to your query you can get an execution plan and estimated costs in miliseconds. The execution plan must be read from bottom to top.

```sql
# Example query with EXPLAIN
EXPLAIN ANALYZE SELECT * FROM customers;

# Output for the query plan
QUERY PLAN
----------
Seq Scan on customers  (cost=0.00..676.00 rows=20000 width=268) (actual time=0.011..34.489 rows=20000 loops=1)
 Total runtime: 65.804 ms
 
# cost=0.00..676.00: 0.00 is the startup estimated cost, how much work estimated to return the first row. 676.00 #total estimated execution cost. 
# rows=20000 the number of rows expected to output
# width=268 average number of bytes per each row output
# actual time=0.11..34.489 the same as cost, but this ones are the real costs of executing the query.
# rows=20000 the actual number of rows that were returned.
# loops=1
```
- A simple way to check if your queries are being fast is avoiding the use of Seq Scan and instead promote the use of Index Scan and Bitmap heap Scan (by adding indivual indexes in the cases describe in the first topic)

3. Aggreation and subsets of tables beforing applying joins.
--------------------------------------------------------------

#### t
