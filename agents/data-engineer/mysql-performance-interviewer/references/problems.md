# Problem Bank: MySQL Performance

## Problem 1: The ESR Index Design Challenge

### Problem Statement
An e-commerce platform has a `orders` table with 80M rows. The most critical query powers the customer order history page:

```sql
SELECT order_id, total_amount, created_at, status
FROM orders
WHERE customer_id = 7891
  AND status IN ('completed', 'shipped')
  AND created_at > '2024-06-01'
ORDER BY created_at DESC
LIMIT 25;
```

Current index: `INDEX(status, created_at, customer_id)`

EXPLAIN shows:
```
+----+------+----------------------------+----------------------------+---------+------+----------+-----------------------------+
| id | type | possible_keys              | key                        | key_len | rows | filtered | Extra                       |
+----+------+----------------------------+----------------------------+---------+------+----------+-----------------------------+
|  1 | range| idx_status_created_cust    | idx_status_created_cust    | 206     | 2.4M | 0.12     | Using where; Using filesort |
+----+------+----------------------------+----------------------------+---------+------+----------+-----------------------------+
```

Query takes 8 seconds. Fix it.

### Solution Walkthrough

**Step 1: Diagnose the current index**
- `status IN ('completed', 'shipped')` — equality (two values, MySQL does index merge or range)
- `created_at > '2024-06-01'` — range
- `customer_id = 7891` — equality (but it's LAST in the index)

The index order `(status, created_at, customer_id)` means:
1. MySQL uses `status` for the IN clause
2. Then `created_at` for range scan — but after this range, the index stops being useful for ordering
3. `customer_id` is in the index but after a range column — it can only be used as a post-filter
4. `Using filesort` — MySQL can't use the index for ORDER BY

**Step 2: Apply ESR Rule**
- **Equality**: `customer_id = 7891` (single value, high cardinality)
- **Sort**: `created_at DESC` (ORDER BY column)
- **Range**: `status IN ('completed', 'shipped')` (IN with 2 values acts as range)

Optimal index:
```sql
ALTER TABLE orders ADD INDEX idx_orders_cust_created_status
  (customer_id, created_at DESC, status);
```

Wait — but `status` is used in the WHERE. Can we do better?

Actually, `IN` with a small number of values can be treated as equality if MySQL can do index merge. But the safest ESR application:

```sql
-- Best index for this specific query:
ALTER TABLE orders ADD INDEX idx_orders_esr
  (customer_id, created_at DESC, status, total_amount, order_id);
```

This is a covering index:
- `customer_id` — equality filter (jump directly to this customer)
- `created_at DESC` — rows come out pre-sorted (no filesort)
- `status` — filter applied within the index
- `total_amount, order_id` — covering (no table lookup needed)

**Step 3: Verify with EXPLAIN**
```
+----+------+----------------+----------------+---------+------+----------+--------------------------+
| id | type | possible_keys  | key            | key_len | rows | filtered | Extra                    |
+----+------+----------------+----------------+---------+------+----------+--------------------------+
|  1 | ref  | idx_orders_esr | idx_orders_esr | 4       | 312  | 45.00    | Using where; Using index |
+----+------+----------------+----------------+---------+------+----------+--------------------------+
```

- `type: ref` — using equality portion of index
- `Using index` — covering index, no table lookup
- `rows: 312` — scanning only this customer's rows
- No `Using filesort` — sort comes from index order
- Result: < 5ms

### Follow-up Questions
- "What if `status` had 20 possible values instead of 2? Would your index change?"
- "The `IN` clause — does MySQL treat it as equality or range? When does it matter?"
- "What's the write cost of this covering index? Is it worth it?"

---

## Problem 2: InnoDB Locking Disaster

### Problem Statement
A Spring Boot application processes order refunds. The endpoint:

```java
@Transactional
public void processRefund(Long orderId) {
    Order order = orderRepo.findById(orderId);           // SELECT ... WHERE id = ?
    PaymentGateway.refund(order.getPaymentId());          // HTTP call: 2-3 seconds
    order.setStatus("refunded");
    order.setRefundedAt(Instant.now());
    orderRepo.save(order);                                // UPDATE ... WHERE id = ?
}
```

At 200 RPS, the `orders` table starts showing lock wait timeouts. Other endpoints that update orders are timing out. The HTTP call to PaymentGateway takes 2-3 seconds.

### Solution Walkthrough

**Root Cause:**
The `@Transactional` annotation wraps the ENTIRE method — including the 2-3 second HTTP call. The sequence:

1. `BEGIN TRANSACTION` (connection acquired from pool)
2. `SELECT ... WHERE id = ?` (shared lock or snapshot read depending on isolation)
3. **HTTP call: 2-3 seconds** (connection held, transaction open)
4. `UPDATE ... WHERE id = ?` (exclusive lock acquired)
5. `COMMIT` (lock released)

The problem: the transaction holds a database connection for 2-3 seconds doing non-DB work. At 200 RPS:
- 200 requests × 2.5s avg hold time = **500 connections needed simultaneously**
- Default HikariCP pool: 10 connections
- Default MySQL `max_connections`: 151
- Result: pool exhaustion → all endpoints blocked

Additionally, if the `SELECT` uses `SELECT ... FOR UPDATE` (or the JPA provider does), it holds an exclusive lock on that row for the entire 2-3 seconds.

**Fix:**
```java
// Move the HTTP call OUTSIDE the transaction
public void processRefund(Long orderId) {
    Order order = orderRepo.findById(orderId);             // No transaction needed for read
    PaymentGateway.refund(order.getPaymentId());           // HTTP call: outside transaction

    updateOrderAsRefunded(orderId);                        // Short transaction: ~5ms
}

@Transactional
public void updateOrderAsRefunded(Long orderId) {
    Order order = orderRepo.findById(orderId);
    order.setStatus("refunded");
    order.setRefundedAt(Instant.now());
    orderRepo.save(order);
}
```

Now the transaction holds the connection for ~5ms instead of ~2500ms.
- 200 RPS × 0.005s = **1 connection** needed on average
- Pool of 10 is more than enough

**Prevention:**
- Rule: NEVER make external HTTP/API calls inside a `@Transactional` method
- Rule: Keep transactions as short as possible — only DB operations
- Add monitoring: alert if avg transaction duration > 100ms
- Configure HikariCP `leak-detection-threshold: 2000` to log long-held connections

### Follow-up Questions
- "What if the refund and the DB update must be atomic? How do you handle the case where the HTTP call succeeds but the DB update fails?"
- "How does `REQUIRES_NEW` propagation affect connection pool usage?"
- "What's the maximum transaction duration you'd accept in a 500 RPS system?"

---

## Problem 3: The Silent Index Killer — Function on Column

### Problem Statement
A reporting query runs nightly but has started timing out as the table grows:

```sql
SELECT DATE(created_at) AS order_date,
       COUNT(*) AS order_count,
       SUM(total_amount) AS revenue
FROM orders
WHERE DATE(created_at) = '2024-03-15'
GROUP BY DATE(created_at);
```

There's an index on `created_at`. EXPLAIN shows:
```
+----+------+---------------+------+---------+------+----------+-------------+
| id | type | possible_keys | key  | key_len | rows | filtered | Extra       |
+----+------+---------------+------+---------+------+----------+-------------+
|  1 | ALL  | NULL          | NULL | NULL    | 80M  | 100.00   | Using where |
+----+------+---------------+------+---------+------+----------+-------------+
```

Full table scan on 80M rows despite having an index. Why?

### Solution Walkthrough

**Root Cause:**
`DATE(created_at)` wraps the indexed column in a function. MySQL cannot use the index because the B-tree is organized by `created_at` (the raw TIMESTAMP/DATETIME value), not by `DATE(created_at)`.

The optimizer sees: "I need to evaluate `DATE(x)` for every row to see if it equals '2024-03-15'. I can't use the index to skip rows because the index doesn't store `DATE(x)` values."

**Fix — Rewrite as range query:**
```sql
SELECT DATE(created_at) AS order_date,
       COUNT(*) AS order_count,
       SUM(total_amount) AS revenue
FROM orders
WHERE created_at >= '2024-03-15 00:00:00'
  AND created_at <  '2024-03-16 00:00:00'
GROUP BY DATE(created_at);
```

Now MySQL can use the index on `created_at` for a range scan — jumping directly to the March 15 entries and stopping at March 16.

**Even better — covering index:**
```sql
ALTER TABLE orders ADD INDEX idx_orders_created_amount
  (created_at, total_amount);
```

With this covering index, MySQL can satisfy the entire query from the index without touching the table data (`Using index` in Extra).

**Common anti-patterns in the same family:**
```sql
-- BAD: function on column breaks index
WHERE YEAR(created_at) = 2024
WHERE LOWER(email) = 'user@example.com'
WHERE customer_id + 0 = 123          -- Yes, even this breaks it

-- GOOD: rewrite to preserve index usage
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
WHERE email = 'user@example.com'     -- Use case-insensitive collation instead
WHERE customer_id = 123
```

**MySQL 8.0+ alternative — functional index (generated column):**
```sql
-- If you MUST filter on a function result:
ALTER TABLE orders ADD COLUMN order_date DATE
  GENERATED ALWAYS AS (DATE(created_at)) STORED;
ALTER TABLE orders ADD INDEX idx_order_date (order_date);

-- Now this works with the index:
WHERE order_date = '2024-03-15'
```

### Follow-up Questions
- "What about implicit type conversions? `WHERE varchar_column = 12345` — does the index work?"
- "When would you use a generated column vs rewriting the query?"
- "How does this interact with prepared statements and parameter binding?"

---

## Problem 4: OR Clause Index Bypass

### Problem Statement
```sql
SELECT order_id, customer_id, total_amount
FROM orders
WHERE customer_id = 789
   OR total_amount > 10000
ORDER BY created_at DESC
LIMIT 50;
```

Indexes exist on both `customer_id` and `total_amount`. EXPLAIN shows a full table scan. Why?

### Solution Walkthrough

**Root Cause:**
The `OR` combines two conditions on different columns. MySQL has two choices:
1. Use `idx_customer_id` to find customer 789's orders, then scan all results for `total_amount > 10000` — misses rows where only `total_amount` matches
2. Use `idx_total_amount` for the amount filter — misses rows where only `customer_id` matches
3. Full table scan — correct but slow

MySQL 5.7+ can sometimes use **index merge (union)**, but the optimizer often decides a full scan is cheaper, especially with ORDER BY on a third column.

**Fix — UNION ALL rewrite:**
```sql
(SELECT order_id, customer_id, total_amount, created_at
 FROM orders
 WHERE customer_id = 789
 ORDER BY created_at DESC
 LIMIT 50)
UNION ALL
(SELECT order_id, customer_id, total_amount, created_at
 FROM orders
 WHERE total_amount > 10000
   AND customer_id != 789  -- Avoid duplicates
 ORDER BY created_at DESC
 LIMIT 50)
ORDER BY created_at DESC
LIMIT 50;
```

Each sub-query uses its own index efficiently. The outer query just merges and re-sorts a maximum of 100 rows.

**Alternative — force index merge (not recommended for production):**
```sql
SELECT /*+ INDEX_MERGE(orders idx_customer_id, idx_total_amount) */
  order_id, customer_id, total_amount
FROM orders
WHERE customer_id = 789 OR total_amount > 10000
ORDER BY created_at DESC
LIMIT 50;
```

### Follow-up Questions
- "When does MySQL actually use index merge? What conditions make it choose full scan instead?"
- "What's the downside of the UNION ALL rewrite?"
- "How would you handle this with 5 OR conditions across different columns?"

---

## Problem 5: Connection Pool Math Under Parallel Calls

### Problem Statement
A backend service handles 500 RPS. Each request makes these DB calls:

```java
CompletableFuture<User> user = CompletableFuture.supplyAsync(() -> userRepo.findById(userId));
CompletableFuture<List<Order>> orders = CompletableFuture.supplyAsync(() -> orderRepo.findByCustomer(userId));
CompletableFuture<Wallet> wallet = CompletableFuture.supplyAsync(() -> walletRepo.findByUser(userId));

CompletableFuture.allOf(user, orders, wallet).join();
```

HikariCP pool size: 50. After a traffic spike to 800 RPS, the service becomes unresponsive. Not just the new endpoint — every endpoint that touches the database.

### Solution Walkthrough

**The Math:**
- 3 parallel DB calls per request
- Each call needs its own connection (they run on separate threads)
- At 500 RPS: 3 × 500 = **1,500 connections needed simultaneously**
- Pool size: 50
- Result: 1,450 threads blocked waiting for a connection

At 800 RPS: 3 × 800 = **2,400 peak connections**. Even MySQL's `max_connections` (default 151) can't handle this.

**Why it kills ALL endpoints:**
HikariCP has a single shared pool. When the parallel-call endpoint exhausts all 50 connections, a simple health-check query `SELECT 1` also can't get a connection. Load balancer marks the instance as unhealthy → cascading failure.

**Fix — layered approach:**

1. **Caching first** (reduce total calls):
```java
// Cache user and wallet lookups — they rarely change
@Cacheable("users")
public User findById(Long id) { ... }

// Now: 3 DB calls → 1 DB call (orders only) for cached users
// 500 RPS × 1 call = 500 connections peak
```

2. **Sequential with early-return** (if caching isn't enough):
```java
User user = userRepo.findById(userId);
if (user == null) return notFound();           // Early return, 1 connection
List<Order> orders = orderRepo.findByCustomer(userId);
Wallet wallet = walletRepo.findByUser(userId);
// 1 connection held sequentially = 500 connections at 500 RPS
```

3. **If parallelism is truly needed — semaphore guard:**
```java
private static final Semaphore DB_SEMAPHORE = new Semaphore(30);

public Result handle(Long userId) {
    DB_SEMAPHORE.acquire();
    try {
        // parallel calls here
    } finally {
        DB_SEMAPHORE.release();
    }
}
// Caps concurrent DB usage to 30, rest queue at the semaphore
```

4. **Separate pools for critical vs batch paths:**
```yaml
# Critical path pool (API requests)
spring.datasource.hikari.maximum-pool-size: 30

# Batch path pool (background jobs)
batch.datasource.hikari.maximum-pool-size: 10
```

### Follow-up Questions
- "How do you size a connection pool? What's the formula?"
- "What happens if a connection is leaked (never returned to pool)?"
- "How does `REQUIRES_NEW` inside an existing transaction affect pool usage?"

---

## Problem 6: Implicit Type Conversion Kills Index

### Problem Statement
```sql
-- Table: users (20M rows)
-- Column: phone VARCHAR(20), indexed
-- Query:
SELECT * FROM users WHERE phone = 8005551234;
```

The index on `phone` exists but EXPLAIN shows `type: ALL` (full table scan). The query takes 15 seconds.

### Solution Walkthrough

**Root Cause:**
`phone` is VARCHAR but the comparison value `8005551234` is an integer. MySQL performs **implicit type conversion** — it converts every `phone` value to an integer for comparison. This is equivalent to:

```sql
WHERE CAST(phone AS UNSIGNED) = 8005551234
```

A function on the indexed column — the index is bypassed.

**Why it's dangerous:**
- No syntax error — the query runs "fine"
- Works correctly on small tables (full scan is fast)
- Only manifests as a problem at scale
- Code review often misses it because it looks correct

**Fix:**
```sql
-- Quote the value — match the column type
SELECT * FROM users WHERE phone = '8005551234';
```

**Other implicit conversion traps:**
```sql
-- DANGEROUS: comparing VARCHAR to INT
WHERE order_code = 12345           -- order_code is VARCHAR

-- DANGEROUS: comparing DATE to string with wrong format
WHERE created_at = '15-03-2024'    -- MySQL may not parse this correctly

-- SAFE: matching types
WHERE order_code = '12345'
WHERE created_at = '2024-03-15'
```

### Follow-up Questions
- "Does the reverse happen? INT column compared to a string — does the index work?"
- "How would you detect implicit conversions in a large codebase?"
- "What role does the MySQL charset/collation play in string comparisons and index usage?"

---

## Sample Session Flow

### Opening (2 minutes)
**Interviewer**: "Hey, I'm the MySQL DBA for this platform. I've been tuning InnoDB since 5.5 and I've seen every way a query can go wrong. Today I want to see how you think about MySQL performance — not just 'make it fast' but 'understand why it's slow.' Let's jump in."

### Phase 1: Warm-up (10 minutes)
**Interviewer**: "Quick one. I have a composite index on `(a, b, c)`. I run `WHERE a = 1 AND c > 10`. How much of the index does MySQL actually use?"

*[Candidate should explain: MySQL uses only column `a`. After `a`, the next indexed column is `b`, which isn't in the WHERE, so the index stops being useful for filtering. Column `c` in the index is unreachable.]*

**Interviewer**: "Good. Now tell me — what's the difference between a clustered index and a secondary index in InnoDB? And why does that matter for query performance?"

### Phase 2: Index Design (20 minutes)
**Interviewer**: "Here's our slowest query — 8 seconds on 50M rows. There IS an index, but it's not helping. Tell me why and fix it."

*[Present Problem 1: ESR Index Design Challenge]*

**Interviewer**: "You reordered the index. Walk me through exactly what MySQL does differently with your new index. Trace the B-tree traversal."

### Phase 3: Production Debugging (20 minutes)
**Interviewer**: "New scenario. After a deploy, every endpoint is returning 504s. The DB CPU is only at 20%. What's your first move?"

*[Present Problem 2 or 5: Locking or Connection Pool]*

**Interviewer**: "You identified the connection pool issue. How do you fix it without just cranking up the pool size?"

### Phase 4: Prevention (10 minutes)
**Interviewer**: "What guardrails would you put in place so this class of problem never hits production again?"

### Closing (3 minutes)
**Interviewer**: "Let me give you some feedback..."

*[Generate scorecard]*
