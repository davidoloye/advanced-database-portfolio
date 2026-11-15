# Portfolio 1 – Database Design

## Objective
Design a relational schema for the fintech system covering **Customers**, **Accounts**, **Transactions**, and **Transaction Logs**. Demonstrate normalization, primary/foreign keys, indexes, and constraints.

## Schema (PostgreSQL)
```sql
-- customers table
CREATE TABLE customers (
    customer_id   BIGSERIAL PRIMARY KEY,
    first_name    VARCHAR(50) NOT NULL,
    last_name     VARCHAR(50) NOT NULL,
    email         VARCHAR(100) UNIQUE NOT NULL,
    phone         VARCHAR(20),
    created_at    TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- accounts table
CREATE TABLE accounts (
    account_id    BIGSERIAL PRIMARY KEY,
    customer_id   BIGINT NOT NULL REFERENCES customers(customer_id),
    account_number VARCHAR(20) UNIQUE NOT NULL,
    account_type  VARCHAR(20) NOT NULL CHECK (account_type IN ('checking','savings','business')),
    balance       NUMERIC(15,2) NOT NULL DEFAULT 0.00,
    created_at    TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- transactions table
CREATE TABLE transactions (
    transaction_id BIGSERIAL PRIMARY KEY,
    from_account   BIGINT NOT NULL REFERENCES accounts(account_id),
    to_account     BIGINT REFERENCES accounts(account_id), -- null for deposits/withdrawals
    amount         NUMERIC(15,2) NOT NULL CHECK (amount > 0),
    txn_type       VARCHAR(20) NOT NULL CHECK (txn_type IN ('transfer','deposit','withdrawal')),
    status         VARCHAR(20) NOT NULL CHECK (status IN ('pending','completed','failed','reversed')),
    created_at     TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- transaction_log table (audit trail)
CREATE TABLE transaction_log (
    log_id         BIGSERIAL PRIMARY KEY,
    transaction_id BIGINT NOT NULL REFERENCES transactions(transaction_id),
    changed_by     VARCHAR(50) NOT NULL, -- e.g., system, user id
    change_action  VARCHAR(30) NOT NULL,
    change_time    TIMESTAMP WITH TIME ZONE DEFAULT now(),
    details        JSONB
);
```

## Indexes for performance
```sql
CREATE INDEX idx_accounts_customer ON accounts(customer_id);
CREATE INDEX idx_txn_from_account ON transactions(from_account);
CREATE INDEX idx_txn_to_account ON transactions(to_account);
CREATE INDEX idx_txn_status ON transactions(status);
```

## Sample Data (fictitious)
```sql
INSERT INTO customers (first_name, last_name, email, phone)
VALUES
    ('Ada','Okafor','ada.okafor@example.com','08012345678'),
    ('Bola','Ibrahim','bola.ih@example.com','08087654321');

INSERT INTO accounts (customer_id, account_number, account_type, balance)
VALUES
    (1, 'NG0010001', 'checking', 15000.00),
    (1, 'NG0010002', 'savings', 30000.00),
    (2, 'NG0020001', 'checking', 5000.00);
```

## Testing the design
1. Run `psql` scripts.
2. Verify foreign‑key constraints by attempting an invalid insert (should fail).
3. Use `EXPLAIN ANALYZE` on sample queries (see portfolio‑2).

---
### Technical Reflection
See `technical-reflection.md` for design decisions, normalization rationale, and challenges.

