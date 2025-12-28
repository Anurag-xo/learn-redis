# Redis Transactions

## What are Transactions?

Transactions in Redis allow you to **execute a group of commands as a single atomic operation** with guaranteed consistency.

---

## Key Guarantees

### 1. **Serialization & Sequential Execution**

All commands in a transaction are executed sequentially without interruption. No other client can execute commands between your transaction commands.

**Benefit:** Prevents race conditions and ensures data consistency

### 2. **Atomicity (All-or-Nothing)**

Either **all commands succeed** or **none of them are applied**.

**Benefit:** Maintains database integrity even if errors occur

---

## Transaction Commands

### MULTI - Start Transaction

Marks the beginning of a transaction block. Subsequent commands are queued instead of executed immediately.

```bash
MULTI
```

**Output:** `OK`

### EXEC - Execute Transaction

Executes all queued commands atomically.

```bash
EXEC
```

**Output:** Array of replies from each command

### DISCARD - Cancel Transaction

Flushes the transaction queue and exits transaction mode without executing any commands.

```bash
DISCARD
```

**Output:** `OK`

### WATCH - Optimistic Locking

Monitors keys for changes. If any watched key is modified before EXEC, the transaction is aborted.

```bash
WATCH key [key ...]
```

---

## Transaction Workflow

```bash
# 1. Start transaction
MULTI
# Output: OK

# 2. Queue commands (not executed yet)
SET account:1:balance 1000
# Output: QUEUED

INCR account:1:transactions
# Output: QUEUED

SET account:1:last_updated "2025-12-28"
# Output: QUEUED

# 3. Execute all commands atomically
EXEC
# Output:
# 1) OK
# 2) (integer) 1
# 3) OK
```

---

## Practical Example: Money Transfer

```bash
# Transfer $100 from account A to account B
MULTI
DECRBY account:A:balance 100
INCRBY account:B:balance 100
SET transfer:status "completed"
EXEC
```

**Result:** All three operations succeed together, or none execute if there's an error.

---

## Using WATCH for Optimistic Locking

Prevents race conditions when multiple clients modify the same data.

```bash
# Client 1
WATCH account:balance
# Output: OK

GET account:balance
# Output: "1000"

# If another client modifies account:balance here,
# the transaction will fail

MULTI
SET account:balance 900
EXEC
# Output: (nil) if key was modified, or array of results if successful
```

**Behavior:**

- If watched key changes before EXEC: transaction aborts, returns `(nil)`
- If no changes detected: transaction executes normally

---

## Error Handling

### Command Queuing Errors

Syntax errors detected during queuing prevent EXEC from running.

```bash
MULTI
SET key value
INVALID_COMMAND
# Output: (error) ERR unknown command

EXEC
# Output: (error) EXECABORT Transaction discarded
```

### Runtime Errors

Commands with correct syntax but runtime failures (e.g., wrong data type) don't abort the transaction.

```bash
MULTI
SET key "string_value"
INCR key  # This will fail at runtime
SET another_key "value"
EXEC
# Output:
# 1) OK
# 2) (error) ERR value is not an integer
# 3) OK
```

⚠️ **Note:** Redis doesn't roll back commands that executed before an error.

---

## Best Practices

✅ **Do:**

- Use transactions for operations that must succeed or fail together
- Use WATCH for conditional transactions based on key values
- Keep transactions short to minimize blocking
- Handle EXEC returning `(nil)` when using WATCH

❌ **Don't:**

- Include slow operations (use pipelining instead)
- Assume automatic rollback on partial failures
- Use transactions for simple single-command operations

---

## MULTI vs Pipelining

| Feature       | MULTI/EXEC                               | Pipelining                       |
| ------------- | ---------------------------------------- | -------------------------------- |
| **Atomicity** | Yes (all-or-nothing)                     | No                               |
| **Ordering**  | Guaranteed sequential                    | Guaranteed sequential            |
| **Use Case**  | Related operations requiring consistency | Batch operations for performance |
| **Rollback**  | Aborts on queue errors                   | No rollback mechanism            |

---

## Common Use Cases

1. **Financial transactions** - Ensuring balance updates are atomic
2. **Counter operations** - Multiple related counters updated together
3. **State changes** - Update multiple fields representing object state
4. **Inventory management** - Decrement stock and create order record
5. **Conditional updates** - Using WATCH to implement optimistic locking

---

## Quick Reference

```bash
# Basic transaction
MULTI
command1
command2
command3
EXEC

# Transaction with optimistic locking
WATCH key
# ... check conditions ...
MULTI
command1
command2
EXEC

# Cancel transaction
MULTI
command1
DISCARD  # Nothing is executed
```
