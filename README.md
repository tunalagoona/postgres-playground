# Postgres Playground

Playing with Postgres concurrency: locks, isolation levels, deadlock prevention.

- [Transaction isolation](/isolation-levels.md#transaction-isolation)
  - [Read committed](/isolation-levels.md#read-committed)
  - [Repeatable read](/isolation-levels.md#repeatable-read)
    
- [Row-level locks](/row-level-locks.md#postgresql-single-locks)
  - [Implicit exclusive row-level lock (ordinary UPDATE)](/row-level-locks.md#implicit-exclusive-row-level-lock-ordinary-update)
  - [Explicit shared row-level lock (SELECT FOR SHARE)](/row-level-locks.md#explicit-shared-row-level-lock-select-for-share)
  - [Explicit exclusive row-level lock (SELECT FOR UPDATE)](/row-level-locks.md#explicit-exclusive-row-level-lock-select-for-update)
    
- [Deadlocks](/deadlocks.md#deadlocks)
  - [Deadlock on implicit exclusive row-level locks (ordinary UPDATE)](/deadlocks.md#deadlock-on-implicit-exclusive-row-level-locks-ordinary-update)
  - [Deadlock on explicit exclusive row-level locks (SELECT FOR UPDATE)](/deadlocks.md#deadlock-on-explicit-exclusive-row-level-locks-select-for-update)

- [Deadlock prevention](/deadlock-prevention.md#deadlock-prevention)
  - [Multiple transactions](/deadlock-prevention.md#multiple-transactions)
  - [Explicit global lock](/deadlock-prevention.md#explicit-global-lock)
  - [Lock hierarchy](/deadlock-prevention.md#lock-hierarchy)


