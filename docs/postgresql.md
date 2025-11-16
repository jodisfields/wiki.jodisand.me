# PostgreSQL Cheatsheet

A comprehensive guide to PostgreSQL - the world's most advanced open-source relational database.

## Table of Contents

- [Installation](#installation)
- [Connection](#connection)
- [Database Operations](#database-operations)
- [Table Operations](#table-operations)
- [Data Types](#data-types)
- [Queries](#queries)
- [Indexes](#indexes)
- [Views](#views)
- [Functions & Procedures](#functions--procedures)
- [Triggers](#triggers)
- [Transactions](#transactions)
- [User Management](#user-management)
- [Backup & Restore](#backup--restore)
- [Performance](#performance)
- [JSON Support](#json-support)

## Installation

### Ubuntu/Debian

```bash
# Add PostgreSQL repository
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Install PostgreSQL
sudo apt update
sudo apt install postgresql-15 postgresql-contrib-15

# Start PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### macOS

```bash
# Using Homebrew
brew install postgresql@15
brew services start postgresql@15
```

### Docker

```bash
# Run PostgreSQL in Docker
docker run --name postgres \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v postgres_data:/var/lib/postgresql/data \
  -d postgres:15

# Docker Compose
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Connection

### psql (Command Line)

```bash
# Connect as postgres user (default superuser)
sudo -u postgres psql

# Connect to specific database
psql -U username -d database_name

# Connect to remote host
psql -h hostname -p 5432 -U username -d database_name

# Connection string
psql postgresql://username:password@hostname:5432/database_name

# Execute SQL file
psql -U username -d database_name -f script.sql

# Execute SQL command
psql -U username -d database_name -c "SELECT * FROM users;"
```

### psql Meta-Commands

```sql
-- List databases
\l
\list

-- Connect to database
\c database_name
\connect database_name

-- List tables
\dt
\dt+  -- With additional info

-- List schemas
\dn

-- List views
\dv

-- Describe table
\d table_name
\d+ table_name  -- With additional info

-- List functions
\df

-- List users/roles
\du

-- List indexes
\di

-- Show current database
\conninfo

-- Execute SQL from file
\i /path/to/file.sql

-- Toggle expanded output
\x

-- Quit
\q

-- Help
\?  -- psql commands
\h  -- SQL commands
\h SELECT  -- Help for specific command
```

## Database Operations

### Create Database

```sql
-- Basic create
CREATE DATABASE mydb;

-- With owner and encoding
CREATE DATABASE mydb
  OWNER myuser
  ENCODING 'UTF8'
  LC_COLLATE = 'en_US.UTF-8'
  LC_CTYPE = 'en_US.UTF-8'
  TEMPLATE template0;

-- With connection limit
CREATE DATABASE mydb
  CONNECTION LIMIT 50;
```

### Drop Database

```sql
-- Drop database
DROP DATABASE mydb;

-- Drop if exists
DROP DATABASE IF EXISTS mydb;

-- Force drop (disconnect active connections)
SELECT pg_terminate_backend(pg_stat_activity.pid)
FROM pg_stat_activity
WHERE pg_stat_activity.datname = 'mydb'
  AND pid <> pg_backend_pid();
DROP DATABASE mydb;
```

### Database Info

```sql
-- List all databases
SELECT datname FROM pg_database;

-- Database size
SELECT pg_size_pretty(pg_database_size('mydb'));

-- All databases with size
SELECT
  datname as database,
  pg_size_pretty(pg_database_size(datname)) as size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;
```

## Table Operations

### Create Table

```sql
-- Basic table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table with constraints
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL,
  total DECIMAL(10, 2) NOT NULL CHECK (total >= 0),
  status VARCHAR(20) DEFAULT 'pending',
  order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  CONSTRAINT fk_user
    FOREIGN KEY (user_id)
    REFERENCES users(id)
    ON DELETE CASCADE,

  CONSTRAINT valid_status
    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
);

-- Table with composite primary key
CREATE TABLE order_items (
  order_id INTEGER NOT NULL,
  product_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  price DECIMAL(10, 2) NOT NULL,

  PRIMARY KEY (order_id, product_id),
  FOREIGN KEY (order_id) REFERENCES orders(id),
  FOREIGN KEY (product_id) REFERENCES products(id)
);
```

### Alter Table

```sql
-- Add column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Add column with default
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT true;

-- Drop column
ALTER TABLE users DROP COLUMN phone;

-- Rename column
ALTER TABLE users RENAME COLUMN username TO user_name;

-- Change column type
ALTER TABLE users ALTER COLUMN email TYPE TEXT;

-- Add constraint
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);

-- Add foreign key
ALTER TABLE orders ADD CONSTRAINT fk_user
  FOREIGN KEY (user_id) REFERENCES users(id);

-- Drop constraint
ALTER TABLE users DROP CONSTRAINT unique_email;

-- Rename table
ALTER TABLE users RENAME TO app_users;

-- Add check constraint
ALTER TABLE products ADD CONSTRAINT price_check CHECK (price >= 0);
```

### Drop Table

```sql
-- Drop table
DROP TABLE users;

-- Drop if exists
DROP TABLE IF EXISTS users;

-- Drop with cascade (remove dependent objects)
DROP TABLE users CASCADE;
```

### Table Info

```sql
-- Table structure
\d table_name

-- Table size
SELECT pg_size_pretty(pg_relation_size('table_name'));

-- Table with indexes size
SELECT pg_size_pretty(pg_total_relation_size('table_name'));

-- All tables with size
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Data Types

### Numeric Types

```sql
-- Integer types
SMALLINT          -- -32768 to 32767
INTEGER / INT     -- -2147483648 to 2147483647
BIGINT            -- -9223372036854775808 to 9223372036854775807
SERIAL            -- Auto-incrementing integer
BIGSERIAL         -- Auto-incrementing bigint

-- Decimal types
DECIMAL(precision, scale)  -- Exact decimal
NUMERIC(precision, scale)  -- Exact decimal (same as DECIMAL)
REAL              -- 6 decimal digits precision
DOUBLE PRECISION  -- 15 decimal digits precision

-- Example
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  price DECIMAL(10, 2),
  weight REAL
);
```

### String Types

```sql
-- Character types
CHAR(n)           -- Fixed length
VARCHAR(n)        -- Variable length with limit
TEXT              -- Variable unlimited length

-- Example
CREATE TABLE documents (
  id SERIAL PRIMARY KEY,
  code CHAR(10),
  title VARCHAR(255),
  content TEXT
);
```

### Date/Time Types

```sql
-- Date/Time types
DATE              -- Date only
TIME              -- Time only
TIMESTAMP         -- Date and time
TIMESTAMPTZ       -- Timestamp with timezone
INTERVAL          -- Time interval

-- Example
CREATE TABLE events (
  id SERIAL PRIMARY KEY,
  event_date DATE,
  event_time TIME,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  starts_at TIMESTAMPTZ,
  duration INTERVAL
);
```

### Boolean Type

```sql
-- Boolean
BOOLEAN           -- TRUE, FALSE, NULL

-- Example
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  is_active BOOLEAN DEFAULT true,
  email_verified BOOLEAN DEFAULT false
);
```

### JSON Types

```sql
-- JSON types
JSON              -- JSON data (stored as text)
JSONB             -- Binary JSON (more efficient)

-- Example
CREATE TABLE settings (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,
  preferences JSONB
);
```

### Array Types

```sql
-- Array types
INTEGER[]
TEXT[]
VARCHAR(50)[]

-- Example
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title TEXT,
  tags TEXT[]
);

-- Insert array
INSERT INTO posts (title, tags) VALUES ('My Post', ARRAY['tech', 'programming']);

-- Query array
SELECT * FROM posts WHERE 'tech' = ANY(tags);
```

### UUID Type

```sql
-- UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- UUID type
CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id INTEGER,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Queries

### SELECT

```sql
-- Basic select
SELECT * FROM users;

-- Select specific columns
SELECT id, username, email FROM users;

-- With WHERE clause
SELECT * FROM users WHERE is_active = true;

-- Multiple conditions
SELECT * FROM users
WHERE is_active = true
  AND created_at > '2024-01-01';

-- LIKE pattern matching
SELECT * FROM users WHERE email LIKE '%@gmail.com';

-- ILIKE (case-insensitive)
SELECT * FROM users WHERE username ILIKE 'john%';

-- IN clause
SELECT * FROM users WHERE id IN (1, 2, 3, 5, 8);

-- BETWEEN
SELECT * FROM orders WHERE total BETWEEN 100 AND 500;

-- IS NULL
SELECT * FROM users WHERE phone IS NULL;

-- ORDER BY
SELECT * FROM users ORDER BY created_at DESC;

-- LIMIT and OFFSET (pagination)
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

-- DISTINCT
SELECT DISTINCT status FROM orders;

-- COUNT
SELECT COUNT(*) FROM users;
SELECT COUNT(DISTINCT email) FROM users;
```

### INSERT

```sql
-- Insert single row
INSERT INTO users (username, email, password_hash)
VALUES ('john', 'john@example.com', 'hashed_password');

-- Insert multiple rows
INSERT INTO users (username, email, password_hash)
VALUES
  ('alice', 'alice@example.com', 'hash1'),
  ('bob', 'bob@example.com', 'hash2');

-- Insert and return
INSERT INTO users (username, email, password_hash)
VALUES ('jane', 'jane@example.com', 'hash3')
RETURNING id, created_at;

-- Insert from SELECT
INSERT INTO archived_users
SELECT * FROM users WHERE created_at < '2020-01-01';

-- Insert with ON CONFLICT (upsert)
INSERT INTO users (id, username, email)
VALUES (1, 'john', 'john@example.com')
ON CONFLICT (id) DO UPDATE
  SET username = EXCLUDED.username,
      email = EXCLUDED.email,
      updated_at = CURRENT_TIMESTAMP;

-- Insert or do nothing
INSERT INTO users (email, username)
VALUES ('test@example.com', 'test')
ON CONFLICT (email) DO NOTHING;
```

### UPDATE

```sql
-- Basic update
UPDATE users SET is_active = false WHERE id = 1;

-- Update multiple columns
UPDATE users
SET
  email = 'newemail@example.com',
  updated_at = CURRENT_TIMESTAMP
WHERE id = 1;

-- Update with calculation
UPDATE products SET price = price * 1.1;

-- Update from another table
UPDATE orders o
SET total = (
  SELECT SUM(quantity * price)
  FROM order_items
  WHERE order_id = o.id
);

-- Update and return
UPDATE users SET is_active = false WHERE id = 1
RETURNING *;
```

### DELETE

```sql
-- Delete rows
DELETE FROM users WHERE id = 1;

-- Delete all rows
DELETE FROM users;

-- Delete with condition
DELETE FROM users WHERE created_at < '2020-01-01';

-- Delete and return
DELETE FROM users WHERE id = 1 RETURNING *;

-- Delete using subquery
DELETE FROM users
WHERE id IN (
  SELECT user_id FROM banned_users
);
```

### JOIN

```sql
-- INNER JOIN
SELECT u.username, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN
SELECT u.username, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username;

-- RIGHT JOIN
SELECT *
FROM orders o
RIGHT JOIN users u ON o.user_id = u.id;

-- FULL OUTER JOIN
SELECT *
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- Multiple joins
SELECT
  u.username,
  o.id as order_id,
  p.name as product_name,
  oi.quantity
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

### Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) FROM users;

-- SUM
SELECT SUM(total) FROM orders;

-- AVG
SELECT AVG(price) FROM products;

-- MIN/MAX
SELECT MIN(price), MAX(price) FROM products;

-- GROUP BY
SELECT status, COUNT(*) as count
FROM orders
GROUP BY status;

-- HAVING
SELECT user_id, COUNT(*) as order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;

-- Multiple aggregates
SELECT
  status,
  COUNT(*) as count,
  SUM(total) as total_amount,
  AVG(total) as avg_amount
FROM orders
GROUP BY status;
```

### Subqueries

```sql
-- Subquery in WHERE
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- Subquery in SELECT
SELECT
  username,
  (SELECT COUNT(*) FROM orders WHERE user_id = users.id) as order_count
FROM users;

-- Subquery in FROM
SELECT avg_price
FROM (
  SELECT AVG(price) as avg_price
  FROM products
  GROUP BY category
) AS category_averages;

-- EXISTS
SELECT * FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

### Common Table Expressions (CTE)

```sql
-- Basic CTE
WITH active_users AS (
  SELECT * FROM users WHERE is_active = true
)
SELECT username FROM active_users;

-- Multiple CTEs
WITH
  high_value_orders AS (
    SELECT * FROM orders WHERE total > 1000
  ),
  vip_users AS (
    SELECT DISTINCT user_id FROM high_value_orders
  )
SELECT u.*
FROM users u
JOIN vip_users v ON u.id = v.user_id;

-- Recursive CTE (organizational hierarchy)
WITH RECURSIVE org_chart AS (
  -- Base case
  SELECT id, name, manager_id, 1 as level
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  -- Recursive case
  SELECT e.id, e.name, e.manager_id, oc.level + 1
  FROM employees e
  JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, id;
```

## Indexes

### Create Index

```sql
-- Basic index
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Composite index
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- B-tree index (default)
CREATE INDEX idx_users_id ON users USING btree(id);

-- Hash index
CREATE INDEX idx_users_email_hash ON users USING hash(email);

-- GIN index (for JSON, arrays, full-text search)
CREATE INDEX idx_posts_tags ON posts USING gin(tags);

-- GiST index (for geometric data)
CREATE INDEX idx_locations ON locations USING gist(coordinates);
```

### Drop Index

```sql
-- Drop index
DROP INDEX idx_users_email;

-- Drop if exists
DROP INDEX IF EXISTS idx_users_email;
```

### Index Info

```sql
-- List indexes
SELECT * FROM pg_indexes WHERE tablename = 'users';

-- Index size
SELECT pg_size_pretty(pg_relation_size('idx_users_email'));

-- Unused indexes
SELECT
  schemaname,
  tablename,
  indexname,
  idx_scan as index_scans
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

## Views

### Create View

```sql
-- Basic view
CREATE VIEW active_users AS
SELECT id, username, email
FROM users
WHERE is_active = true;

-- View with joins
CREATE VIEW user_orders AS
SELECT
  u.username,
  u.email,
  o.id as order_id,
  o.total,
  o.status
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- Materialized view (cached results)
CREATE MATERIALIZED VIEW sales_summary AS
SELECT
  DATE_TRUNC('month', order_date) as month,
  COUNT(*) as order_count,
  SUM(total) as total_sales
FROM orders
GROUP BY DATE_TRUNC('month', order_date);

-- Refresh materialized view
REFRESH MATERIALIZED VIEW sales_summary;

-- Concurrent refresh (doesn't lock view)
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
```

### Drop View

```sql
-- Drop view
DROP VIEW active_users;

-- Drop materialized view
DROP MATERIALIZED VIEW sales_summary;
```

## Functions & Procedures

### Create Function

```sql
-- Simple function
CREATE OR REPLACE FUNCTION get_user_count()
RETURNS INTEGER AS $$
BEGIN
  RETURN (SELECT COUNT(*) FROM users);
END;
$$ LANGUAGE plpgsql;

-- Use function
SELECT get_user_count();

-- Function with parameters
CREATE OR REPLACE FUNCTION get_orders_by_user(p_user_id INTEGER)
RETURNS TABLE(order_id INTEGER, total DECIMAL, status VARCHAR) AS $$
BEGIN
  RETURN QUERY
  SELECT id, total, status
  FROM orders
  WHERE user_id = p_user_id;
END;
$$ LANGUAGE plpgsql;

-- Use function
SELECT * FROM get_orders_by_user(5);

-- Function with default parameters
CREATE OR REPLACE FUNCTION search_users(
  p_query TEXT DEFAULT '',
  p_limit INTEGER DEFAULT 10
)
RETURNS TABLE(id INTEGER, username VARCHAR, email VARCHAR) AS $$
BEGIN
  RETURN QUERY
  SELECT u.id, u.username, u.email
  FROM users u
  WHERE u.username ILIKE '%' || p_query || '%'
    OR u.email ILIKE '%' || p_query || '%'
  LIMIT p_limit;
END;
$$ LANGUAGE plpgsql;
```

### Stored Procedures

```sql
-- Procedure (no return value)
CREATE OR REPLACE PROCEDURE deactivate_old_users(p_days INTEGER)
LANGUAGE plpgsql AS $$
BEGIN
  UPDATE users
  SET is_active = false
  WHERE last_login < CURRENT_DATE - p_days;

  COMMIT;
END;
$$;

-- Call procedure
CALL deactivate_old_users(90);
```

### Drop Function

```sql
-- Drop function
DROP FUNCTION get_user_count();

-- Drop procedure
DROP PROCEDURE deactivate_old_users(INTEGER);
```

## Triggers

### Create Trigger

```sql
-- Trigger function
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger
CREATE TRIGGER users_updated_at
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

-- Audit trigger
CREATE TABLE audit_log (
  id SERIAL PRIMARY KEY,
  table_name VARCHAR(50),
  operation VARCHAR(10),
  old_data JSONB,
  new_data JSONB,
  changed_by VARCHAR(100),
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION audit_changes()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    INSERT INTO audit_log (table_name, operation, new_data, changed_by)
    VALUES (TG_TABLE_NAME, TG_OP, row_to_json(NEW), current_user);
    RETURN NEW;
  ELSIF TG_OP = 'UPDATE' THEN
    INSERT INTO audit_log (table_name, operation, old_data, new_data, changed_by)
    VALUES (TG_TABLE_NAME, TG_OP, row_to_json(OLD), row_to_json(NEW), current_user);
    RETURN NEW;
  ELSIF TG_OP = 'DELETE' THEN
    INSERT INTO audit_log (table_name, operation, old_data, changed_by)
    VALUES (TG_TABLE_NAME, TG_OP, row_to_json(OLD), current_user);
    RETURN OLD;
  END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_audit
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW
EXECUTE FUNCTION audit_changes();
```

### Drop Trigger

```sql
-- Drop trigger
DROP TRIGGER users_updated_at ON users;
```

## Transactions

### Basic Transactions

```sql
-- Begin transaction
BEGIN;

-- Execute queries
INSERT INTO users (username, email) VALUES ('test', 'test@example.com');
UPDATE orders SET status = 'shipped' WHERE id = 100;

-- Commit transaction
COMMIT;

-- Rollback transaction
ROLLBACK;
```

### Savepoints

```sql
BEGIN;

INSERT INTO users (username, email) VALUES ('user1', 'user1@example.com');

SAVEPOINT sp1;

INSERT INTO users (username, email) VALUES ('user2', 'user2@example.com');

-- Rollback to savepoint
ROLLBACK TO SAVEPOINT sp1;

COMMIT;
```

### Transaction Isolation Levels

```sql
-- Read Uncommitted (not supported in PostgreSQL, defaults to Read Committed)
-- Read Committed (default)
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Repeatable Read
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Serializable
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

## User Management

### Create User/Role

```sql
-- Create user
CREATE USER myuser WITH PASSWORD 'mypassword';

-- Create user with options
CREATE USER myuser WITH
  PASSWORD 'mypassword'
  CREATEDB
  CREATEROLE
  LOGIN;

-- Create role (no login)
CREATE ROLE readonly;
```

### Grant Privileges

```sql
-- Grant database privileges
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
GRANT CONNECT ON DATABASE mydb TO myuser;

-- Grant table privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON users TO myuser;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO myuser;

-- Grant schema privileges
GRANT USAGE ON SCHEMA public TO myuser;

-- Grant sequence privileges (for SERIAL columns)
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO myuser;

-- Grant execute on functions
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO myuser;

-- Make user owner
ALTER DATABASE mydb OWNER TO myuser;
ALTER TABLE users OWNER TO myuser;

-- Grant role to user
GRANT readonly TO myuser;
```

### Revoke Privileges

```sql
-- Revoke privileges
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM myuser;
REVOKE SELECT ON users FROM myuser;
```

### Alter User

```sql
-- Change password
ALTER USER myuser WITH PASSWORD 'newpassword';

-- Rename user
ALTER USER myuser RENAME TO newuser;

-- Set connection limit
ALTER USER myuser CONNECTION LIMIT 10;
```

### Drop User

```sql
-- Drop user
DROP USER myuser;

-- Reassign ownership before dropping
REASSIGN OWNED BY myuser TO postgres;
DROP OWNED BY myuser;
DROP USER myuser;
```

## Backup & Restore

### pg_dump (Backup)

```bash
# Backup entire database
pg_dump mydb > mydb_backup.sql
pg_dump -U username mydb > mydb_backup.sql

# Backup in custom format (compressed)
pg_dump -Fc mydb > mydb_backup.dump

# Backup specific tables
pg_dump mydb -t users -t orders > tables_backup.sql

# Backup only schema
pg_dump --schema-only mydb > schema_backup.sql

# Backup only data
pg_dump --data-only mydb > data_backup.sql

# Backup with compression
pg_dump mydb | gzip > mydb_backup.sql.gz

# Backup all databases
pg_dumpall > all_databases_backup.sql
```

### pg_restore (Restore)

```bash
# Restore from SQL file
psql mydb < mydb_backup.sql
psql -U username -d mydb < mydb_backup.sql

# Restore from custom format
pg_restore -d mydb mydb_backup.dump

# Restore with options
pg_restore -d mydb --clean --if-exists mydb_backup.dump

# Restore specific tables
pg_restore -d mydb -t users mydb_backup.dump

# Restore with parallel jobs
pg_restore -d mydb -j 4 mydb_backup.dump
```

## Performance

### EXPLAIN

```sql
-- Explain query plan
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Explain with execution
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Detailed explain
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT * FROM users WHERE email = 'test@example.com';
```

### VACUUM

```sql
-- Vacuum table (reclaim space)
VACUUM users;

-- Vacuum and analyze
VACUUM ANALYZE users;

-- Full vacuum (locks table, more thorough)
VACUUM FULL users;

-- Vacuum entire database
VACUUM;

-- Auto-vacuum is enabled by default
SHOW autovacuum;
```

### ANALYZE

```sql
-- Update statistics
ANALYZE users;

-- Analyze entire database
ANALYZE;
```

### Performance Queries

```sql
-- Slow queries
SELECT
  query,
  calls,
  total_time,
  mean_time,
  max_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Table sizes
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;

-- Index usage
SELECT
  schemaname,
  tablename,
  indexname,
  idx_scan,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Cache hit ratio
SELECT
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit)  as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;

-- Active connections
SELECT
  count(*),
  state
FROM pg_stat_activity
GROUP BY state;

-- Long-running queries
SELECT
  pid,
  now() - pg_stat_activity.query_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY duration DESC;
```

## JSON Support

### JSON Operations

```sql
-- Create table with JSONB
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  attributes JSONB
);

-- Insert JSON data
INSERT INTO products (name, attributes)
VALUES (
  'Laptop',
  '{"brand": "Dell", "specs": {"ram": "16GB", "cpu": "i7"}, "tags": ["electronics", "computers"]}'
);

-- Query JSON field
SELECT * FROM products WHERE attributes->>'brand' = 'Dell';

-- Query nested JSON
SELECT * FROM products WHERE attributes->'specs'->>'ram' = '16GB';

-- Query JSON array
SELECT * FROM products WHERE attributes->'tags' ? 'electronics';

-- JSON operators
-- -> Get JSON object field (returns JSON)
SELECT attributes->'brand' FROM products;

-- ->> Get JSON object field (returns text)
SELECT attributes->>'brand' FROM products;

-- #> Get JSON object at path
SELECT attributes#>'{specs,ram}' FROM products;

-- #>> Get JSON object at path (returns text)
SELECT attributes#>>'{specs,ram}' FROM products;

-- @> Contains
SELECT * FROM products WHERE attributes @> '{"brand": "Dell"}';

-- ? Key exists
SELECT * FROM products WHERE attributes ? 'brand';

-- ?| Any key exists
SELECT * FROM products WHERE attributes ?| array['brand', 'model'];

-- ?& All keys exist
SELECT * FROM products WHERE attributes ?& array['brand', 'specs'];
```

### JSON Functions

```sql
-- json_build_object
SELECT json_build_object('name', name, 'attributes', attributes)
FROM products;

-- jsonb_set (update JSON)
UPDATE products
SET attributes = jsonb_set(attributes, '{specs,ram}', '"32GB"')
WHERE id = 1;

-- jsonb_insert
UPDATE products
SET attributes = jsonb_insert(attributes, '{new_field}', '"value"')
WHERE id = 1;

-- jsonb_pretty
SELECT jsonb_pretty(attributes) FROM products;

-- jsonb_array_elements
SELECT jsonb_array_elements(attributes->'tags') as tag
FROM products;

-- jsonb_each
SELECT * FROM jsonb_each((SELECT attributes FROM products WHERE id = 1));
```

### JSON Index

```sql
-- GIN index for JSON
CREATE INDEX idx_products_attributes ON products USING gin(attributes);

-- Index specific JSON path
CREATE INDEX idx_products_brand ON products((attributes->>'brand'));
```

## Additional Resources

- [Official PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [Postgres Guide](http://postgresguide.com/)
- [PostgreSQL Exercises](https://pgexercises.com/)

---

*Last updated: 2025-11-16*
