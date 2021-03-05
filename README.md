# Postgres Playground

Playing with Postgres concurrency: locks, isolation levels, deadlock prevention.

- [Transaction isolation](/isolation-levels.md#transaction-isolation)
  - [Read committed](/isolation-levels.md#read-committed)
  - [Repeatable read](/isolation-levels.md#repeatable-read)
    
- [Single row-level lock](/single-lock.md#postgresql-single-locks)
  - [Implicit exclusive row-level lock (ordinary update)](/single-lock.md#implicit-exclusive-row-level-lock-ordinary-update)
  - [Explicit shared row-level lock (select for share)](/single-lock.md#explicit-shared-row-level-lock-select-for-share)
  - [Explicit exclusive row-level lock (select for update)](/single-lock.md#explicit-exclusive-row-level-lock-select-for-update)
    
- [Deadlocks](/deadlocks.md#deadlocks)
  - [Deadlock on implicit exclusive row-level locks](/deadlocks.md#deadlock-on-implicit-exclusive-row-level-locks)
  - [Deadlock on explicit exclusive row-level locks](/deadlocks.md#deadlock-on-explicit-exclusive-row-level-locks)

- [Deadlock prevention](/deadlock-prevention.md#deadlock-prevention)
  - [Multiple transactions](/deadlock-prevention.md#multiple-transactions)
  - [Explicit global lock](/deadlock-prevention.md#explicit-global-lock)
  - [Lock hierarchy](/deadlock-prevention.md#lock-hierarchy)


