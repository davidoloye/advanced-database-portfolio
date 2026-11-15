# Portfolio 4 – NoSQL (MongoDB)

## Objective
Illustrate modelling the same fintech domain using a document‑oriented database. Show how to store customers, accounts, and transactions in MongoDB collections, query patterns, and how to enforce data integrity at the application layer.

## Data Model (MongoDB)
- **customers** – `{ _id, firstName, lastName, email, phone, createdAt }`
- **accounts** – `{ _id, customerId, accountNumber, type, balance, createdAt }`
- **transactions** – `{ _id, fromAccount, toAccount, amount, type, status, createdAt }`
- **transactionLogs** – `{ _id, transactionId, changedBy, action, time, details }`

We embed the **transactionLogs** inside the `transactions` document for fast audit access, but keep separate collections for reporting.

## Sample Commands (mongo shell)
```js
// Insert customers
db.customers.insertMany([
  { firstName: 'Ada', lastName: 'Okafor', email: 'ada.okafor@example.com', phone: '08012345678', createdAt: new Date() },
  { firstName: 'Bola', lastName: 'Ibrahim', email: 'bola.ih@example.com', phone: '08087654321', createdAt: new Date() }
]);

// Create accounts referencing customers
var ada = db.customers.findOne({ email: 'ada.okafor@example.com' });
var bola = db.customers.findOne({ email: 'bola.ih@example.com' });

db.accounts.insertMany([
  { customerId: ada._id, accountNumber: 'NG0010001', type: 'checking', balance: 15000, createdAt: new Date() },
  { customerId: ada._id, accountNumber: 'NG0010002', type: 'savings', balance: 30000, createdAt: new Date() },
  { customerId: bola._id, accountNumber: 'NG0020001', type: 'checking', balance: 5000, createdAt: new Date() }
]);

// Transfer funds (application‑level transaction)
function transfer(fromAccNum, toAccNum, amount, initiator) {
  const session = db.getMongo().startSession();
  session.startTransaction();
  try {
    const fromAcc = db.accounts.findOne({ accountNumber: fromAccNum }, { session });
    const toAcc   = db.accounts.findOne({ accountNumber: toAccNum },   { session });
    if (fromAcc.balance < amount) throw 'Insufficient funds';

    db.accounts.updateOne({ _id: fromAcc._id }, { $inc: { balance: -amount } }, { session });
    db.accounts.updateOne({ _id: toAcc._id },   { $inc: { balance:  amount } }, { session });

    const txn = db.transactions.insertOne({
      fromAccount: fromAcc._id,
      toAccount:   toAcc._id,
      amount,
      type: 'transfer',
      status: 'completed',
      createdAt: new Date()
    }, { session });

    db.transactionLogs.insertOne({
      transactionId: txn.insertedId,
      changedBy: initiator,
      action: 'transfer',
      time: new Date(),
      details: { amount }
    }, { session });

    session.commitTransaction();
    print('Transfer succeeded');
  } catch (e) {
    session.abortTransaction();
    print('Transfer failed:', e);
  } finally {
    session.endSession();
  }
}

transfer('NG0010001', 'NG0020001', 2000, 'system');
```

## Query Optimisation
- Create indexes on `accountNumber`, `customerId`, and `transactions.fromAccount`/`toAccount`.
```js
db.accounts.createIndex({ accountNumber: 1 }, { unique: true });
db.accounts.createIndex({ customerId: 1 });
db.transactions.createIndex({ fromAccount: 1 });
db.transactions.createIndex({ toAccount: 1 });
```
- Use covered queries (project only indexed fields) to avoid fetching full documents.

## Testing & Validation
1. Run the script above and verify balances via `db.accounts.find()`.
2. Use `explain('executionStats')` on a sample aggregation pipeline to view index usage.
3. Capture the `explain` output and store in `logs/explain_transfer.json`.

## Supporting Artifacts
- **screenshots/** – screenshot of MongoDB Compass showing collections and indexes.
- **logs/** – `explain_transfer.json`, transaction logs.
- **config/** – `mongo-init.js` used for initial data load.

---
### Technical Reflection
See `technical-reflection.md` in this folder for notes on schema design choices, handling of ACID guarantees with MongoDB sessions, and trade‑offs compared to the relational approach.

