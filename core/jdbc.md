# [←](../README.md) <a id="home"></a> JDBC

## Table of content:
- [JDBC](#jdbc)
- [ACID](#acid)
- [Transactions](#trx)
- [Consistency](#consistency)
- [Tables](#tbl)
- [Normal Forms](#normal)
- [Joins](#joins)
- [Groups](#groups)
- [Indexes](#indexes)
- [Explain](#explain)
- [NoSQL](#nosql)

----

## [↑](#Home) <a name="jdbc"></a> JDBC
**JDBC** stands for **Java Database Connectivity**.\
**JDBC** is the standard Java API that enables applications to connect, query, and interact with relational databases and other tabular data sources.

JDBC is a standard to relational databases. So it means that NOSQL databases uses different approaches because they use different data structures.\
But for relational databases most frameworks based on the JDBC. That's why it's important to understand SQL and JDBC.

Different databases uses different API to communicate with databases. Because database is a separate server application.\
To adapt the same API (i.e. JDBC API) to different databases the JDBC drivers are used.

The JDBC driver can be downloaded from the official website OR it's better to use build tools like Maven or Gradle.\
Coordinates can be found in the database documentation OR at the **[Maven Repository](https://mvnrepository.com/)** page.
 
Previously it was needed to call class to load JDBC Driver class and trigger static initializers:
```java
Class.forName("com.mysql.jdbc.Driver");
```

But after JDBC 4.0+ the JDBC Driver loading happens with **SPI (Service Provider Interface)**.\
JDBC Driver jar contains specific metadata in the specific file ``META-INF/services/java.sql.Driver``.\
JDBC DriverManager knows that it should scan such files and load JDBC drivers that are mentioned there.

**SQL (Structured Query Language)** is a language that can perform two types of operations:
- DDL (Data Definition Language): change database structure (or change metadata, like CREATE, ALTER, DROP, TRUNCATE)
- DML (Data Manipulation Language): change data inside the database  (or change data, like INSERT, UPDATE, DELETE, SELECT)

Driver Manager is used to get a connection to the database:
```java
Connection conn = DriverManager.getConnection(
	"jdbc:postgresql://localhost:5432/test",
	"user",
	"password"
);
```

The connection object can be used to create and perform SQL operations:
```java
Statement stmt = conn.createStatement();
```
or 
```java 
PreparedStatement ps =
    conn.prepareStatement(
        "INSERT INTO users(name, age) VALUES (?, ?)"
    );
```
The difference is that regular statement is created and not cached or handled in any specific way.\
It's parsed each time again. And usually such appoach is used when statement should be executed only once and doesn't have any dynamic data.\
Such statements are quite dangerous because the SQL injections can be used with them.

Prepared statements handles input arguments that are passed via methods:
```java
ps.setString(1, "Alex");
ps.setInt(2, 25);
```
That's why it's more secure way to deal with SQL statements.

When we can call ``executeQuery()`` to execute the SELECT statement and get the ResultSet.\
OR we can call ``executeUpdate()`` to call DDL operations (like INSERT, UPDATE, DELETE) and get the count of changed rows.

For example:
```java
ResultSet rs = ps.executeQuery();
while (rs.next()) {
    System.out.println(
        rs.getInt("id") + " " +
        rs.getString("name")
    );
}
```

Also, we should keep in mind that ResultSet and Statements are not just a Java objects.\
They are resources that should be closed.

For example, Statement can store the sql query state on the JDBC driver side.\
For PreparedState it can hold the prepared SQL query execution plan.\
Also, the cursor can be holded and reference to JDBC connection.

The same for ResultSet. It holds the cursor on the database side, data buffers, connection to the database and other resources.\
So we always should use try-with-resources with them.

Also, we should keep in mind that JDBC covers mainly infrastructure API.\
It means that JDBC doesn't unify the SQL specifics of different databases.\
SQL is a standard but it's quite flexible and different databases implements SQL standard in a different way.\
Also, there are different SQL standard versions.

For example, the ID generation language depends on the SQL standard version and on the SQL database.\
SQL query example for PostgreSQL:
```sql
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username VARCHAR(50) NOT NULL
);
```
For more details: **"[PostgreSQL documentation: Identity Columns](https://www.postgresql.org/docs/current/ddl-identity-columns.html)"**.

As we can see, JDBC is a quite low-level mechanism to communicate with database.\
That's why where are different frameworks on top of it: Hibernate, JOOQ, Spring Data, MyBatis and other.

----

## [↑](#Home) <a name="acid"></a> ACID
**ACID** is an acronym for Atomicity, Consistency, Isolation, and Durability.\
These four core properties guarantee that database transactions are processed safely and reliably, even during system crashes, power failures, or simultaneous user updates.

The 4 Pillars of ACID:
- **A**tomicity: "All or nothing" execution
- **C**onsistency: Moves data from one valid state to another
- **I**solation: Transactions do not interfere with each other
- **D**urability: Committed data is permanently saved

**Durability** topic is more about Database implementations.\
But it can be interesting to know how it works and sometimes it can be even very useful.

For example, PostgreSQL database uses **WAL**.\
**WAL** stands for Write-Ahead Logging. The idea is to write information about changes effectively to the specific log (on commit).\
And then this information will be applied later by special background writer.\
It allows to do the WAL replay on database start to replay changes that was missed for some reason (like electricity outage, etc). 

MySQL database uses a little bit similar approach with **Redo Log**.\
The durability mechanism behavior can be tweaked by ``innodb_flush_log_at_trx_commit`` settings for innodb MySQL engine.

Oracle is also quite similar. It uses **Redo Log** in memory and a specific process to write it to the disc.

**A**tomicity is implemented by **transactions** mechanism.

----

## [↑](#Home) <a name="trx"></a> Transactions
**Transaction** is a Unit Of Work in terms of database.\
All operations inside the transactions can be committed (i.e. approved) only all of them OR none of them.\
I.e. the main idea is "all or nothing".

From the JDBC perspective Transaction is a part of Connection and not presented as a separate object.\
It means that Connection should be used to configure transactions.

For example, how to demarcate transactions? (i.e. how to define transaction boundaries)\
The **commit** ends the transaction.

By default, JDBC enables autocommit mode for transactions.\
But it can be changed:
```java
conn.setAutoCommit(false);
```
In that case ony manual commit finalize the transaction: ``conn.commit();``.\
New transaction will be opened on the next sql statement execution.

Transactions can be isolated from other transactions to control the data consistency AND to achieve a balance of performance.\
The transaction isolation level can be set for the connection:
```java 
conn.setTransactionIsolation(DEFAULT_LEVEL);
conn.setAutoCommit(false);
```
Transaction isolation controls WHICH DATA is visible FROM this transaction.\
It means that transaction CAN'T control how OTHER transaction sees own data.

There are several [isolation levels](https://docs.oracle.com/cd/E19830-01/819-4721/beamv/index.html):
- TRANSACTION_READ_UNCOMMITTED: our transaction can see uncommitted changes from other Transactions
- TRANSACTION_READ_COMMITTED: our transaction can see everything that was committed by other Transactions
- TRANSACTION_REPEATABLE_READ: our transaction can see data ONLY committed before the transaction began
- TRANSACTION_SERIALIZABLE: emulates serial transaction execution for all committed transactions

For more information: **"[PostgreSQL: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)"**.\
How transactions isolation works depends on the database.

For example, PostgreSQL uses **MVCC (Multi-Version Concurrency Control)**.\
Each database row has ``xmin`` and ``xmax`` hidden columns to track transactions that can see this row.\
That's why READ_UNCOMMITTED is not available for PostgreSQL at all and works as READ_COMMITTED.

Other databases (like MySQL and Oracle) mix MVCC with other tools.\
For example, MySQL mixes MVCC with locks and Oracle mixes MVCC with "undo segments".

Why we need different isolation levels?\
They address different "anomalies," but this has different impacts on performance.

Possible anomalies are:
- dirty read:\
A transaction reads data written by a concurrent uncommitted transaction.
- nonrepeatable read:\
A transaction re-reads data it has previously read and finds that data has been modified by another transaction (that committed since the initial read)
- phantom read:\
A transaction re-executes a query returning a set of rows that satisfy a search condition and finds that the set of rows satisfying the condition has changed due to another recently-committed transaction.
- serialization anomaly:\
The result of successfully committing a group of transactions is inconsistent with all possible orderings of running those transactions one at a time.

Also, Locks may be used:
- FOR UPDATE
```sql
SELECT balance
FROM accounts
WHERE id = 1
FOR UPDATE;
```
Row lock anyway will be held till the transaction end
- FOR SHARE
```sql
SELECT *
FROM orders
WHERE created_at >= '2024-01-01'
FOR SHARE;
```
Row lock will be held IF row will be modified by other transactions

Modern frameworks like Hibernate can use optimistic locking (``@Version``) instead of FOR UPDATE:
```java
@Entity
public class User {
    @Id
    private Long id;
    @Version
    private long version;
    private int balance;
}
```
In that case the exception will be thrown if version will be changed by other transactions commits.

Pessimistic locks can be configured. For example, in Hibernate:
```java
User user = entityManager.find(
    User.class, 1L, LockModeType.PESSIMISTIC_WRITE
);
```

----

## [↑](#Home) <a name="consistency"></a> Consistency
**Consistency** - it's about maintaining data in a correct (i.e. consistent) state.

To restrict incorrect actions with data, **constraints** can be used.

The most used one is **PRIMARY KEY**:
```sql
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT UNIQUE
);
```
Primary keys are used to uniquely identify records in a database.

**UNIQUE** is another useful constraint.\
There is only one Primary key.\
When we need to add uniqueness feature the unique constraint can be used.\
Database creates a specific index to increase the search speed because we need to check uniqueness on each addition.

**NOT NULL** constraint prevents NULL data definition:
```sql
name TEXT NOT NULL
```
We can't insert NULL in such database column.

**DEFAULT** constraint allows to provide default data when column hasn't data at insertion time:
```sql
created_at TIMESTAMP DEFAULT now()
```

**CHECK** constraint allows to make more complex checks (more complex than just not null):
```sql
price NUMERIC CHECK(price > 0)
```

**FOREIGN KEY** allows to connect columns in different tables.\
It prevents from referencing to absent (incorrect) data:
```sql
CREATE TABLE users (
    id INT PRIMARY KEY
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```
We can't put value to ``user_id`` inside ``orders`` if this ID is absent in ``users`` table.

By default, Foreign Key has the option: ``ON DELETE RESTRICT``.\
It means that By default restrict user delition because it has dependent orders.\
But there are different options:
- ``FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE``\
Delete all related orders when user is deleted
- ON DELETE SET NULL\
Set column value to null for all dependent rows instead of deleted entry
- ON DELETE SET DEFAULT\
Set default value for all dependent rows instead of deleted entry

Different databases provides different ways to analyze constraints.\
For example, for PostgreSQL the **[Information Schema](https://www.postgresql.org/docs/current/information-schema.html)** can be used:
```sql
SELECT * FROM information_schema.table_constraints WHERE table_name = 'employees';
```
The **[aiven.io pg playground](https://aiven.io/tools/pg-playground)** can be used for testing.

Also, keep in mind that **Primary key** not always is present:
```sql
CREATE TABLE employees(age INT, name TEXT NOT NULL);
```
In that case PostgreSQL uses for own needs internal identifier ``ctid`` (other databases have own mechanism).\
But it's a bad practice and should be avoided.

Primary Keys can be **composite**.\
For example:
```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id)
);
```

Primary key can be used as:
- **surrogate key**:\
Artificial identifier like 1, 2, 3, ... without any domain (i.e. real world) meaning
- **natural key**:\
A key with domain meaning like email, ISBN, etc

----

## [↑](#Home) <a name="tbl"></a> Tables
Information is databases is stored inside tables.\
And tables can have different types.

**Regular (or heap)** tables is a default table type.\
For example:
```sql 
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```
This is Permanent table. It means that it will be stored even after connections/transactions end.

Another type is **Temporary tables**.\
Such tables are temporary tables and will be deleted when connection is closed.\
For example:
```sql
CREATE TEMP TABLE session_data (
    user_id INT,
    token TEXT
);
```
There can be several options to control when to delete data inside the save connection BUT between different transactions:
- ON COMMIT DROP: drop table on commit
- ON COMMIT DELETE ROWS: save table but delete all rows on commit
- ON COMMIT PRESERVE ROWS: save all data on commit 

Also, there is another type of table: **VIEW**.\
VIEW tables represent a view on a data that is expressed by SQL query.\
For example:
```sql
CREATE VIEW active_users AS
SELECT id, name
FROM users
WHERE active = true;
```
Such "tables" are stored in the system catalog of the PostgreSQL (as a schema object).\
It means that they are stored even after connections/transactions end.\
But they can be deleted in a similar way as any regular table:
```sql
DROP VIEW active_users;
```

There is also another type of view tables: **MATERIALIZED VIEW**.\
Such tables work as a snapshot. It means that they store information that is a result of a query at the moment of query execution, i.e. table creation.\
For example:
```sql 
CREATE MATERIALIZED VIEW user_stats AS
SELECT user_id, COUNT(*) AS orders_count
FROM orders
GROUP BY user_id;
```

To update the materialized view a specific statement should be used:
```sql
REFRESH MATERIALIZED VIEW user_stats;
```
To not lock the whole table with multiple concurrent threads the concurrent refresh can be used:
```sql 
REFRESH MATERIALIZED VIEW CONCURRENTLY user_stats;
```

Also, another interesting type of tables is **Partitioned tables**.\
Table can be created:
```sql 
CREATE TABLE orders (
    id INT,
    created_at DATE,
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (created_at);
```
Primary Key must include the range column.

And then the partition should be created:
```sql
CREATE TABLE orders_2025 PARTITION OF orders
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

There are different types of partitioning:
- Range Partitioning: ``PARTITION BY RANGE (order_date)``\
Use case:
```sql
CREATE TABLE orders_2025
PARTITION OF orders
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```
- List Partitioning: ``PARTITION BY LIST (country)``\
Use case:
```sql
CREATE TABLE users_europe
PARTITION OF users
FOR VALUES IN ('Germany', 'France', 'Serbia');
```
- Hash Partitioning: ``PARTITION BY HASH (id)``\
Use case:
```sql
CREATE TABLE customers_p0
PARTITION OF customers
FOR VALUES WITH (MODULUS 4, REMAINDER 0);
```
- Default Partition:\
```sql
CREATE TABLE orders_other
PARTITION OF orders
DEFAULT;
```

Also, partitions can be combined:
```sql
CREATE TABLE orders_2026
PARTITION OF orders
FOR VALUES FROM ('2026-01-01')
TO ('2027-01-01')
PARTITION BY LIST(region);

CREATE TABLE orders_2026_europe
PARTITION OF orders_2026
FOR VALUES IN ('Europe');
```

Indexes should be created on the main, parent table. It will be propagated to all partitions.

Other databases can have own types of tables.\
For example, Oracle has index-organized tables.\
Such tables don't have heap tables and fully stored in table PK index table.\
It allows to do a really fast search, but it's really difficult to update such table or to have several indexes or do a range scan.

----

## [↑](#Home) <a name="normal"></a> Normal Forms
**SQL normal forms** are rules to create a database architecture.\
They help to reduce data duplication and minimize anomalies for more consistent data changes.

Normal forms are quite intuitive, but anyway they have more formal description:
- 1NF — First Normal Form\
Should not have lists or different data inside the same column (like phones, contacts, etc).\
- 2NF — Second Normal Form\
1NF and: all columns must depend on the whole Primary Key (especially when the PK is composite).\
For example, in the row ``order_id | product_id | product_name | quantity`` the ``product_name`` depends ONLY on ``product_id``, not from the whole pair.
- 3NF — Third Normal Form\
There should not be non-relative columns. For example:\
``employee_id | department_id | department_name``, department_name doesn't depend on employee
- 4NF — Fourth Normal Form\
There should not be rows like this:
```sql
Ivan    | Chess | English
Ivan    | Chess | German
```
- 5NF — Fifth Normal Form
It is impossible to expand the table further without losing information.

**The main idea:**\
stick to common sense and don't store different data in the same table.

Sometimes we need to step aside these rules. Such process is called **denormalization**.

----

## [↑](#Home) <a name="joins"></a> Joins
When we stick to normal forms we have several tables with different data.\
Quite often we need to get data from several tables at the same time.\
In that case the **join** should be used.

The structure of **joins** is quite simple:
```sql
SELECT *
FROM users u INNER JOIN orders o
ON u.id = o.user_id;
```

Joining works **ON** condition.\
The reaction to this condition depends on the type of connection (i.e. the connection type influences the reaction).

There are different types of joining:
- **INNER**: returns Left and Right ONLY when Left and Right rows are present\
This join type is used when we need to get ONLY linked data. For example, orders of a particular user:
```sql
SELECT u.name, o.amount
FROM users u JOIN orders o
ON u.id = o.user_id;
```
Also, usually inner join is a default join type and the inner keyword can be omitted.

- **LEFT JOIN**: return Left row always, right has NULL if right is absent (or value)\
Such join is useful when we need to see if one entry has something related or not:
```sql
SELECT u.name, o.amount
FROM users u LEFT JOIN orders o
ON u.id = o.user_id;
```

- **RIGHT JOIN**: return Right row always, left has NULL if left is absent (or value)\
Right join can be quite useful when we need to check the data consistency. For example:
```sql
SELECT u.name, o.amount
FROM users u RIGHT JOIN orders o
ON u.id = o.user_id;
```
We don't expect orders WITHOUT user.

- **FULL JOIN**: like Left + Right joins together\
Can be quite useful for audit purpose and to compare two sources of information:
```sql 
SELECT u.name, o.amount
FROM users u FULL OUTER JOIN orders o
ON u.id = o.user_id;
```

- **CROSS JOIN**: Cartesian product\
Used to create variations. For example: color x size, city x product, etc.\
Must be used with EXTREME caution (due to the explosive growth of rows).

- **SELF JOIN**: Any join with the same Table\
Can be used when table store relations/hierarchy:
```sql
SELECT 
    e.name AS employee,
    m.name AS manager
FROM employees e LEFT JOIN employees m
ON e.manager_id = m.id;
```
This is the join type that should be chosen! Because we would like to see all employees regardless of whether employee has a manager.
  
Also, the inner join can be written in **"implicit join"** style:
```sql
SELECT u.name, o.amount
FROM users u, orders o
WHERE u.id = o.user_id;
```
SQL optimizer produces the same result as regular inner join.

----

## [↑](#Home) <a name="groups"></a> Groups
Groups is a powerful feature for SQL.\
Rows can be groupped to call aggregate functions.

For example:
```sql
SELECT user_id, COUNT(*) AS total_orders
FROM orders
GROUP BY user_id;
```
user_id is a group. Inside the group we have multiple rows. We can't show them. But we can use them to call aggregate functions.

Also, not all groups can be included into the result.\
Groups can be filtered by the ``HAVING`` keyword:
```sql
SELECT user_id, COUNT(*) AS cnt
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

Also, it's important to remember that aggregate function ignore NULL (except COUNT(*)).\
Also, SELECT keyword is handled AFTER the HAVING. It means that we can't use aliases in the HAVING clause.

HAVING can use different aggregate functions OR different fields.\
Because we already have a group and can analyze (i.e. do aggregation) in different ways.

----

## [↑](#Home) <a name="queries"></a> Queries
SQL queries can be written in different ways.

At first, we should remember **nested subquery**.\
Nested queries are queries that are inserted in the main query.\
They can be inserted in SELECT, WHERE or FROM clauses.

For example:
```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

Nested queries in FROM clause example:
```sql 
SELECT dept_id, avg_salary
FROM (
    SELECT department_id AS dept_id,
           AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_stats;
```

Nested queries are possible even in the SELECT clause:
```sql
SELECT name,
       (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;
```
But it should be used VERY carefully because it can be triggered for each row.

⚠️ Use ``EXPLAIN`` or ``EXPLAIN ANALYZE`` to check the performance!


**Common Table Expressions (CTE)** is another type of queries.\
They allow to create named query result:
```sql 
WITH avg_salary AS (
    SELECT AVG(salary) AS avg_sal
    FROM employees
)
SELECT *
FROM employees
WHERE salary > (SELECT avg_sal FROM avg_salary);
```
In that case we can find employees with with above-average salaries.

There can be several CTEs:
```sql
WITH high_salary AS (
    SELECT * FROM employees WHERE salary > 5000
),
it_department AS (
    SELECT * FROM high_salary WHERE department_id = 1
)
SELECT * FROM it_department;
```

Also, CTE allows to define **hierarchical queries**:
```sql 
WITH RECURSIVE cte_name (column1, column2, ...)
AS(
    -- anchor member
    SELECT select_list FROM table1 WHERE condition
    UNION [ALL]
    -- recursive term
    SELECT select_list FROM cte_name WHERE recursive_condition
)
SELECT * FROM cte_name;
```
For more information: **"[PostgreSQL Recursive Query](https://neon.com/postgresql/tutorial/recursive-query)"**.

Also, several queries can be combined together (their results).\
The ``UNION`` or ``UNION ALL`` should be used in such cases:
```sql
SELECT name FROM employees

UNION ALL

SELECT name FROM contractors;
```
UNION ALL just combines results, but UNION eleminates duplicates.\
Also, queries should have the same count of columns.

----

## [↑](#Home) <a name="indexes"></a> Indexes
**Index** in SQL databases is a specific datastructure to simplify the database search and improve search speed.

Because index is an additional datastructure it means that they consume additional space on the disk.\
Also, because on each table update we need to update related indexes it also take resources.\
That's why it's important to use indexes ONLY when they are really needed.

There are different indexes types. They are also depend on the database implementation.\
Basic types are:
- B-Tree (default index)
```sql 
CREATE INDEX idx ON table(column);
```
- Hash index (only for == operation)
- GIN (Generalized Inverted Index) for arrays, jsonb and fulltext search
```sql
CREATE INDEX idx ON docs USING GIN (data);
```
- GiST index for geo data
- BRIN index for enormous tables with ordered data

For example, Oracle database has a **bitmap index** that is absent in PostgreSQL.\
This index uses bit matrix to represents rows with low cardinality values (columns that can have very limited variety of data).\
Like:
```
M: 1 0 1
F: 0 1 0
```
For female and male values.

Also, Oracle has **reverse key index** that reverse keys bytes.\
It helps to improve index performance for sequential data.

Also, the data selectivity is important for indexes.\
If it's not selective (i.e. has two values like F and M), it's not very efficient to use index.\
Because index has a reference to the structure where table data is stored.\
It means that we always need jump between index and real table structure.\
For big tables it will be a problem.

Indexes are good, but sometimes it's important also HOW we get data:
```sql
SELECT *
FROM users
WHERE id IN (SELECT user_id FROM orders);
```
Or 
```sql
SELECT *
FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```
It depends on the SQL optimizer. But we always should keep in mind different ways to get data.

Also, we can restrict data amount (for example, get the second page):
```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 50 OFFSET 50;
```

Also, results can be cached by frameworks like Spring, Hibernate, etc.

----

## [↑](#Home) <a name="explain"></a> Explain
EXPLAIN command can explain how SQL query WILL BE executed (or EXPLAIN ANALYZE to really check how it's executed).

For testing purposes we can use any online PostgreSQL Playgrounds.\
For example: **[pgtutorial playground](https://www.pgtutorial.com/playground/)**.

For example, we have a Table ``transactions``:
```sql
EXPLAIN SELECT * FROM transactions;
```

Returned information example:
```
Seq Scan on transactions (cost=10000000000.00..10000000024.50 rows=1450 width=28)
```
As we can see, the **Sequential Scan (Seq Scan)** is used (i.e. full scan).\
Because we can't use any indexes, we would like to get all rows. So we have no reasons to use any indexes.

But we get a row by Primary Key column filter:
```sql 
EXPLAIN SELECT * FROM transactions WHERE transaction_id=1;
```
In that case the **Index Scan** will be used.\
If we use columns without index the **Seq scan** will be used instead.\
To fix it we need to add index. For example:
```sql
CREATE INDEX transactions_idx ON transactions(user_id);
```
After that filtering on user_id field will use the index scan.

More interesting thing is JOIN explaining.\
For example:
```sql
EXPLAIN SELECT p.product_id, p.product_name, b.brand_name
  FROM products p INNER JOIN brands b ON p.brand_id = b.brand_id;
```

We can see something like this:
```
Hash Join (rows=250 width=738)
Hash Cond: (p.brand_id = b.brand_id)
-> Seq Scan on products p (rows=250 width=226)
-> Hash (rows=140 width=520)
-> Index Scan using brands_pkey on brands b (rows=140 width=520)
```
As we can see, this JOIN query will be executed (or at least planned to be executed) with **HASH JOIN**.\
PostgreSQL creat a hash table with values from the smallest table (``brands`` in our case).\
Then, the sequential scan on product will be done and rows will be mapped to the hash table rows.

Another type of joining is: **Merge Join (Sort-Merge Join)**.\
For example:
```sql 
EXPLAIN SELECT * FROM users u JOIN profiles p using (user_id);
```
In that case PostgreSQL sort both tables. Both tables have one row per user.\
It means that we can sort both tables and then go one by one to map tables rows.

Another type of joining is **Nested Loop Join**:
```sql
EXPLAIN SELECT
    c.category_name AS category,
    p.product_name AS product
FROM categories c
JOIN products p
    ON p.category_id = c.category_id
WHERE c.category_id = 1;
```
When index for ``products.category_id`` is there, it's worth to use Nested Loop.\
PostgreSQL decides that iterate over categories is cheap. That's why Nested Loop is used.\
If PostgreSQL thinks that it can be expensive, than other join methods (like Hash Join) will be used.

Also, there can be used **Bitmap Heap Scan**.\
In that case instead of sequence scan or index scan, the specific algorithms is used:
- search rows through the index
- create bitmap with matching rows
- group rows by pages in the database
- read them group by group, sequentially when on the same page

It helps to reduce jumping between different pages that can be expensive.

----

## [↑](#Home) <a name="nosql"></a> NoSQL
SQL databases provide JDBC drivers to work with them.\
NOSQL databases usually have own NON-JDBC drivers.

There are different NOSQL databases.\
For example:
- Document NOSQL databases: MongoDB\
Store data as documents (JSON/BSON)
```json
{
  "user": "Alex",
  "orders": [1,2,3]
}
```
Useful for flexible data structure, unstable requirements, scaling.\
But it's difficult to use with complex relations in data. Also, data duplication is possible.

- Graph databases: Neo4j\
Data presented as nodes and edges.\
Can be useful when data can be presented as graphs. For example: routes, relations, links, etc.\
Not convenient for simple tables.

- Key-Value stores: Redis\
Usually used for caches, sessions and other data that should be accessible by the key.

----


