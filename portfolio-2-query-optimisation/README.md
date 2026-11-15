# Portfolio 2 – Query Optimisation

## Objective
Show how to tune queries for the fintech schema, compare naive vs. indexed queries, and capture execution plans.

## Sample Queries
```sql
-- 1. Retrieve all transactions for a given account in the last 30 days
SELECT *
FROM transactions
WHERE from_account = 1
  AND created_at >= now() - interval '30 days';

-- 2. Summarise total deposits per customer
SELECT c.customer_id, c.first_name, c.last_name,
       SUM(t.amount) AS total_deposits
FROM customers c
JOIN accounts a ON a.customer_id = c.customer_id
JOIN transactions t ON t.to_account = a.account_id
WHERE t.txn_type = 'deposit' AND t.status = 'completed'
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_deposits DESC;
```

## Optimisation Steps
1. **Create covering indexes** for the columns used in `WHERE` and `JOIN` clauses.
   ```sql
   CREATE INDEX idx_txn_from_account_created ON transactions(from_account, created_at DESC);
   CREATE INDEX idx_txn_to_account_type_status ON transactions(to_account, txn_type, status);
   ```
2. **Use `EXPLAIN (ANALYZE, BUFFERS)`** to view the planner's choice.
   ```sql
   EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM transactions WHERE from_account = 1 AND created_at >= now() - interval '30 days';
   ```
3. **Rewrite queries** to enable index‑only scans, e.g., select only needed columns.
   ```sql
   SELECT transaction_id, amount, status, created_at
   FROM transactions
   WHERE from_account = 1
     AND created_at >= now() - interval '30 days';
   ```

## Sample Execution Plans (excerpt)
```
Seq Scan on transactions  (cost=0.00..1250.00 rows=5000 width=240) (actual time=0.012..15.432 rows=4500 loops=1)
  Filter: ((from_account = 1) AND (created_at >= (now() - '30 days'::interval)))
  Rows Removed by Filter: 12000
```
After the index:
```
Index Scan using idx_txn_from_account_created on transactions  (cost=0.42..22.84 rows=500 width=240) (actual time=0.004..0.215 rows=4500 loops=1)
  Index Cond: ((from_account = 1) AND (created_at >= (now() - '30 days'::interval)))
```

## Testing the Optimisation
1. Run the queries **before** creating indexes and capture `EXPLAIN ANALYZE` output.
2. Create the indexes.
3. Re‑run the queries and compare execution time & buffer usage.
4. Document the before/after logs in the `logs/` folder of this portfolio.

---
### Technical Reflection
See `technical-reflection.md` for design decisions, indexing trade‑offs, and lessons learned about PostgreSQL's planner.

