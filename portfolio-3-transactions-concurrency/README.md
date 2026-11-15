# Portfolio 3 – Transactions & Concurrency

## Objective
Demonstrate safe fund transfers, reversal of failed transactions, and audit of account balances using ACID‑compliant transactions. Show how different isolation levels affect concurrency and consistency.

## Setup (PostgreSQL)
```bash
psql -d fintech -f setup.sql   # contains the schema from portfolio‑1
```

## Sample Transaction Procedure (PL/pgSQL)
```sql
CREATE OR REPLACE FUNCTION transfer_funds(
    p_from_account BIGINT,
    p_to_account   BIGINT,
    p_amount       NUMERIC,
    p_initiator    VARCHAR)
RETURNS VOID AS $$
DECLARE
    v_from_balance NUMERIC;
    v_to_balance   NUMERIC;
    v_txn_id       BIGINT;
BEGIN
    -- start a transaction block
    PERFORM pg_advisory_xact_lock(p_from_account);
    PERFORM pg_advisory_xact_lock(p_to_account);

    SELECT balance INTO v_from_balance FROM accounts WHERE account_id = p_from_account FOR UPDATE;
    IF v_from_balance < p_amount THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;

    SELECT balance INTO v_to_balance FROM accounts WHERE account_id = p_to_account FOR UPDATE;

    -- debit source
    UPDATE accounts SET balance = balance - p_amount WHERE account_id = p_from_account;
    -- credit destination
    UPDATE accounts SET balance = balance + p_amount WHERE account_id = p_to_account;

    -- record transaction
    INSERT INTO transactions (from_account, to_account, amount, txn_type, status)
    VALUES (p_from_account, p_to_account, p_amount, 'transfer', 'completed')
    RETURNING transaction_id INTO v_txn_id;

    -- audit log
    INSERT INTO transaction_log (transaction_id, changed_by, change_action, details)
    VALUES (v_txn_id, p_initiator, 'transfer', jsonb_build_object('amount', p_amount));

EXCEPTION WHEN OTHERS THEN
    -- on any error, reverse partial changes and mark failed
    RAISE NOTICE 'Transaction failed, rolling back';
    PERFORM pg_advisory_xact_unlock(p_from_account);
    PERFORM pg_advisory_xact_unlock(p_to_account);
    INSERT INTO transactions (from_account, to_account, amount, txn_type, status)
    VALUES (p_from_account, p_to_account, p_amount, 'transfer', 'failed');
    RAISE;
END;
$$ LANGUAGE plpgsql;
```

## Concurrency Experiments
| Isolation Level | Outcome (simulated two concurrent transfers) |
|-----------------|----------------------------------------------|
| **READ COMMITTED** | Possible **lost update** if advisory locks are omitted. |
| **REPEATABLE READ** | Prevents lost update, but phantom rows can appear when new accounts are added. |
| **SERIALIZABLE** | Guarantees full serializability; one transaction aborts with `serialization_failure`. |

Run the following script to spawn two concurrent transfers and capture the logs:
```bash
python3 concurrency_demo.py
```
(The script uses `psycopg2` with `isolation_level=psycopg2.extensions.ISOLATION_LEVEL_SERIALIZABLE`.)

## Testing Steps
1. Populate sample accounts (see portfolio‑1 data).
2. Execute `SELECT * FROM accounts;` before and after the transfers.
3. Verify balances match `SUM(deposits) – SUM(withdrawals) + net_transfers`.
4. Inspect `transaction_log` for the audit trail.
5. Compare execution durations under each isolation level (recorded in `logs/`).

---
### Technical Reflection
See `technical-reflection.md` for notes on lock granularity, dead‑lock handling, and why we chose advisory locks over `SELECT … FOR UPDATE` alone.

