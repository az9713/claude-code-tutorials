---
name: database
description: Database design patterns, queries, and optimization
---

# Database Design & Optimization

## Schema Design Principles

### Normalization vs Denormalization

```sql
-- Normalized (3NF) - Good for writes, data integrity
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL
);

CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  total DECIMAL(10,2) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE order_items (
  id UUID PRIMARY KEY,
  order_id UUID REFERENCES orders(id),
  product_id UUID REFERENCES products(id),
  quantity INT NOT NULL,
  price DECIMAL(10,2) NOT NULL
);

-- Denormalized - Good for reads, reporting
CREATE TABLE order_summaries (
  id UUID PRIMARY KEY,
  user_id UUID,
  user_email VARCHAR(255),      -- Duplicated for fast access
  user_name VARCHAR(255),       -- Duplicated
  total DECIMAL(10,2),
  item_count INT,
  created_at TIMESTAMPTZ
);
```

### Indexing Strategies

```sql
-- Primary key (automatic index)
CREATE TABLE users (
  id UUID PRIMARY KEY
);

-- Unique constraint (creates index)
ALTER TABLE users ADD CONSTRAINT users_email_unique UNIQUE (email);

-- Regular index for frequent lookups
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Composite index (order matters!)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
-- Good for: WHERE user_id = ? AND status = ?
-- Good for: WHERE user_id = ?
-- NOT good for: WHERE status = ?

-- Partial index (subset of rows)
CREATE INDEX idx_orders_pending ON orders(created_at)
WHERE status = 'pending';

-- Expression index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
```

## Query Optimization

### Explain Analyze

```sql
-- Always check query plans for slow queries
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;

-- Look for:
-- - Seq Scan (full table scan) on large tables
-- - High cost estimates
-- - Loops with many iterations
```

### N+1 Query Problem

```typescript
// ❌ N+1 queries
const users = await db.query('SELECT * FROM users');
for (const user of users) {
  user.orders = await db.query('SELECT * FROM orders WHERE user_id = ?', [user.id]);
}

// ✅ Single query with JOIN
const usersWithOrders = await db.query(`
  SELECT u.*, json_agg(o.*) as orders
  FROM users u
  LEFT JOIN orders o ON o.user_id = u.id
  GROUP BY u.id
`);

// ✅ Or two queries with IN
const users = await db.query('SELECT * FROM users');
const userIds = users.map(u => u.id);
const orders = await db.query('SELECT * FROM orders WHERE user_id = ANY(?)', [userIds]);
```

### Pagination

```sql
-- Offset pagination (simple but slow for large offsets)
SELECT * FROM products
ORDER BY created_at DESC
LIMIT 20 OFFSET 1000;  -- Scans 1020 rows!

-- Cursor pagination (efficient for large datasets)
SELECT * FROM products
WHERE created_at < '2024-01-15T10:30:00Z'
ORDER BY created_at DESC
LIMIT 20;

-- Keyset pagination (most efficient)
SELECT * FROM products
WHERE (created_at, id) < ('2024-01-15T10:30:00Z', 'last-id')
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

## Common Patterns

### Soft Deletes

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) NOT NULL,
  deleted_at TIMESTAMPTZ,  -- NULL = not deleted

  -- Unique constraint only on non-deleted
  CONSTRAINT users_email_unique UNIQUE (email) WHERE deleted_at IS NULL
);

-- Always filter in queries
SELECT * FROM users WHERE deleted_at IS NULL;

-- Or use a view
CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;
```

### Audit Trail

```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  table_name VARCHAR(255) NOT NULL,
  record_id UUID NOT NULL,
  action VARCHAR(50) NOT NULL,  -- INSERT, UPDATE, DELETE
  old_data JSONB,
  new_data JSONB,
  user_id UUID,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Trigger function
CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit_log (table_name, record_id, action, old_data, new_data, user_id)
  VALUES (
    TG_TABLE_NAME,
    COALESCE(NEW.id, OLD.id),
    TG_OP,
    CASE WHEN TG_OP = 'DELETE' THEN row_to_json(OLD) ELSE NULL END,
    CASE WHEN TG_OP != 'DELETE' THEN row_to_json(NEW) ELSE NULL END,
    current_setting('app.current_user_id', true)::UUID
  );
  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;
```

### Optimistic Locking

```sql
CREATE TABLE products (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  version INT NOT NULL DEFAULT 1  -- Version column
);

-- Update with version check
UPDATE products
SET price = 29.99, version = version + 1
WHERE id = '123' AND version = 5;
-- If no rows affected, someone else updated it
```

## Migrations Best Practices

```sql
-- Always wrap in transactions
BEGIN;

-- Add columns as nullable first
ALTER TABLE users ADD COLUMN phone VARCHAR(50);

-- Backfill data
UPDATE users SET phone = '' WHERE phone IS NULL;

-- Then add constraints
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;

COMMIT;

-- For large tables, use batches
DO $$
DECLARE
  batch_size INT := 1000;
BEGIN
  LOOP
    UPDATE users
    SET phone = ''
    WHERE id IN (
      SELECT id FROM users
      WHERE phone IS NULL
      LIMIT batch_size
    );
    EXIT WHEN NOT FOUND;
    COMMIT;
  END LOOP;
END $$;
```
