# Redis String Commands

## Overview

Strings are the most basic and versatile data type in Redis. They are binary-safe and can contain any kind of data (text, integers, serialized objects, images) up to 512MB in size.

---

## Basic String Commands

### SET - Store a String Value

Stores a string value at the specified key.

**Syntax:** `SET key value [EX seconds] [PX milliseconds] [NX|XX]`

```bash
SET user:name "Anurag Kumar"
```

**Output:** `OK`

**Options:**

- `EX seconds` - Set expiration in seconds
- `PX milliseconds` - Set expiration in milliseconds
- `NX` - Only set if key doesn't exist
- `XX` - Only set if key already exists

```bash
SET session:token "abc123" EX 3600
# Sets token with 1 hour expiration
```

### GET - Retrieve a String Value

Returns the value stored at the specified key.

**Syntax:** `GET key`

```bash
GET user:name
```

**Output:** `"Anurag Kumar"`

If key doesn't exist:
**Output:** `(nil)`

---

## Bulk Operations

### MSET - Set Multiple Keys

Sets multiple key-value pairs in a single atomic operation.

**Syntax:** `MSET key1 value1 key2 value2 [key3 value3 ...]`

```bash
MSET user:1:name "Alice" user:1:email "alice@example.com" user:1:age "25"
```

**Output:** `OK`

**Benefits:**

- Faster than multiple SET commands
- Atomic operation
- Reduces network round trips

### MGET - Get Multiple Keys

Retrieves values of multiple keys in a single operation.

**Syntax:** `MGET key1 key2 [key3 ...]`

```bash
MGET user:1:name user:1:email user:1:age
```

**Output:**

```
1) "Alice"
2) "alice@example.com"
3) "25"
```

If a key doesn't exist, returns `(nil)` for that position:

```bash
MGET user:1:name user:1:phone user:1:email
```

**Output:**

```
1) "Alice"
2) (nil)
3) "alice@example.com"
```

### MSETNX - Set Multiple Keys (Only if None Exist)

Sets multiple keys only if **all** of them don't exist. It's an atomic operation.

**Syntax:** `MSETNX key1 value1 key2 value2 [key3 value3 ...]`

```bash
MSETNX product:1:name "Laptop" product:1:price "999"
```

**Output:** `(integer) 1` _(success - all keys were set)_

If any key already exists:

```bash
MSETNX product:1:name "Desktop" product:2:name "Monitor"
```

**Output:** `(integer) 0` _(failed - at least one key exists, none were set)_

---

## Expiration Commands

### SETEX - Set with Expiration (Seconds)

Sets a key with a value and expiration time in seconds. Atomic operation.

**Syntax:** `SETEX key seconds value`

```bash
SETEX session:abc123 3600 "user_data"
# Key expires after 1 hour (3600 seconds)
```

**Output:** `OK`

**Equivalent to:**

```bash
SET session:abc123 "user_data" EX 3600
```

### PSETEX - Set with Expiration (Milliseconds)

Sets a key with a value and expiration time in milliseconds.

**Syntax:** `PSETEX key milliseconds value`

```bash
PSETEX rate_limit:user:123 5000 "10"
# Key expires after 5 seconds (5000 milliseconds)
```

**Output:** `OK`

**Use case:** Fine-grained rate limiting, temporary locks

### TTL - Time to Live (Seconds)

Returns the remaining time to live of a key in seconds.

**Syntax:** `TTL key`

```bash
SETEX temp:data 120 "value"
TTL temp:data
```

**Output:** `(integer) 120`

After 30 seconds:

```bash
TTL temp:data
```

**Output:** `(integer) 90`

**Return values:**

- Positive integer: seconds until expiration
- `-1`: key exists but has no expiration
- `-2`: key does not exist

### PTTL - Time to Live (Milliseconds)

Returns the remaining time to live of a key in milliseconds.

**Syntax:** `PTTL key`

```bash
PSETEX lock:resource 5000 "locked"
PTTL lock:resource
```

**Output:** `(integer) 4987`

**Return values:** Same as TTL but in milliseconds

---

## String Manipulation

### APPEND - Append to String

Appends a value to an existing string. If key doesn't exist, creates it.

**Syntax:** `APPEND key value`

```bash
SET message "Hello"
APPEND message " World"
```

**Output:** `(integer) 11` _(new length of string)_

```bash
GET message
```

**Output:** `"Hello World"`

If key doesn't exist:

```bash
APPEND newkey "First"
```

**Output:** `(integer) 5`

### STRLEN - Get String Length

Returns the length of the string stored at key.

**Syntax:** `STRLEN key`

```bash
SET description "Redis is awesome"
STRLEN description
```

**Output:** `(integer) 16`

For non-existent keys:

```bash
STRLEN nonexistent
```

**Output:** `(integer) 0`

### GETRANGE - Get Substring

Returns a substring of the string stored at key.

**Syntax:** `GETRANGE key start end`

```bash
SET text "Hello Redis World"
GETRANGE text 0 4
```

**Output:** `"Hello"`

```bash
GETRANGE text 6 10
```

**Output:** `"Redis"`

**Negative indices** count from the end:

```bash
GETRANGE text -5 -1
```

**Output:** `"World"`

Get entire string:

```bash
GETRANGE text 0 -1
```

**Output:** `"Hello Redis World"`

---

## Advanced String Commands

### GETSET - Get Old Value and Set New

Atomically sets a new value and returns the old value.

**Syntax:** `GETSET key value`

```bash
SET counter "10"
GETSET counter "20"
```

**Output:** `"10"` _(returns old value)_

```bash
GET counter
```

**Output:** `"20"` _(new value is set)_

**Use case:** Rotating tokens, updating counters while retrieving old values

If key doesn't exist:

```bash
GETSET newkey "value"
```

**Output:** `(nil)`

### RENAME - Rename a Key

Renames a key. Overwrites destination if it exists.

**Syntax:** `RENAME oldkey newkey`

```bash
SET user:temp:1 "Alice"
RENAME user:temp:1 user:permanent:1
```

**Output:** `OK`

```bash
GET user:permanent:1
```

**Output:** `"Alice"`

```bash
GET user:temp:1
```

**Output:** `(nil)` _(old key no longer exists)_

**Error handling:**

```bash
RENAME nonexistent newkey
```

**Output:** `(error) ERR no such key`

### RENAMENX - Rename Only if New Key Doesn't Exist

Renames a key only if the new key doesn't already exist.

**Syntax:** `RENAMENX oldkey newkey`

```bash
SET key1 "value1"
SET key2 "value2"
RENAMENX key1 key2
```

**Output:** `(integer) 0` _(failed - key2 already exists)_

```bash
RENAMENX key1 key3
```

**Output:** `(integer) 1` _(success - key3 didn't exist)_

---

## Database Management

### FLUSHALL - Clear All Databases

Deletes all keys from all databases on the server.

**Syntax:** `FLUSHALL [ASYNC]`

```bash
FLUSHALL
```

**Output:** `OK`

**⚠️ WARNING:** This is a destructive operation that cannot be undone!

**Async option** (recommended for large databases):

```bash
FLUSHALL ASYNC
```

Deletes keys in the background without blocking the server.

**Related commands:**

- `FLUSHDB` - Deletes all keys in current database only
- `FLUSHDB ASYNC` - Async version of FLUSHDB

---

## Practical Examples

### Example 1: User Session Management

```bash
# Create session with 30-minute expiration
SETEX session:xyz789 1800 "user_id:42,role:admin"

# Check remaining time
TTL session:xyz789
# Output: (integer) 1795

# Retrieve session data
GET session:xyz789
# Output: "user_id:42,role:admin"
```

### Example 2: Caching API Response

```bash
# Cache API response for 5 minutes
SET cache:api:users:list "[{id:1,name:'Alice'},{id:2,name:'Bob'}]" EX 300

# Check if cache exists and get value
GET cache:api:users:list

# Check time remaining
TTL cache:api:users:list
# Output: (integer) 287
```

### Example 3: Building User Profile

```bash
# Set multiple profile fields at once
MSET user:123:name "John Doe" user:123:email "john@example.com" user:123:country "USA"

# Retrieve all profile fields
MGET user:123:name user:123:email user:123:country
# Output:
# 1) "John Doe"
# 2) "john@example.com"
# 3) "USA"
```

### Example 4: Append Log Entries

```bash
# Initialize log
SET log:app:errors ""

# Append error messages
APPEND log:app:errors "[2025-12-28 10:30] Connection timeout\n"
# Output: (integer) 38

APPEND log:app:errors "[2025-12-28 10:31] Database error\n"
# Output: (integer) 71

# Retrieve full log
GET log:app:errors
```

### Example 5: Atomic Counter Rotation

```bash
# Get old counter value and reset to 0
SET daily:visits 1523
GETSET daily:visits 0
# Output: "1523" (yesterday's count retrieved, today starts at 0)
```

### Example 6: Extract Partial Data

```bash
# Store JSON-like string
SET product:data "SKU:ABC123|Name:Laptop|Price:999|Stock:50"

# Extract just the SKU
GETRANGE product:data 0 9
# Output: "SKU:ABC123"

# Extract price
GETRANGE product:data 27 36
# Output: "Price:999"
```

---

## Command Comparison Table

| Command    | Purpose                       | Atomicity            | Returns                 |
| ---------- | ----------------------------- | -------------------- | ----------------------- |
| `SET`      | Set single key                | Yes                  | OK                      |
| `GET`      | Get single key                | N/A                  | String value            |
| `MSET`     | Set multiple keys             | Yes                  | OK                      |
| `MGET`     | Get multiple keys             | N/A                  | Array of values         |
| `MSETNX`   | Set multiple if none exist    | Yes (all-or-nothing) | 1 (success) or 0 (fail) |
| `SETEX`    | Set with expiration (seconds) | Yes                  | OK                      |
| `PSETEX`   | Set with expiration (ms)      | Yes                  | OK                      |
| `TTL`      | Time to live in seconds       | N/A                  | Integer                 |
| `PTTL`     | Time to live in milliseconds  | N/A                  | Integer                 |
| `APPEND`   | Append to string              | Yes                  | New string length       |
| `STRLEN`   | Get string length             | N/A                  | Integer                 |
| `GETRANGE` | Get substring                 | N/A                  | Substring               |
| `GETSET`   | Get old, set new              | Yes                  | Old value               |
| `RENAME`   | Rename key                    | Yes                  | OK                      |

---

## Best Practices

✅ **Do:**

- Use `MSET`/`MGET` for batch operations to reduce network overhead
- Set appropriate expiration times for temporary data using `SETEX`/`PSETEX`
- Use `MSETNX` for initialization that should only happen once
- Check `TTL` before accessing potentially expired keys
- Use `GETSET` for atomic value rotation

❌ **Don't:**

- Use `FLUSHALL` in production without extreme caution
- Store very large strings (>100KB) if possible; consider compression
- Forget to set expiration on temporary data (leads to memory bloat)
- Use `APPEND` for frequently changing data; use other data structures instead

---

## Quick Reference

```bash
# Basic operations
SET key "value"
GET key
DEL key

# Bulk operations
MSET key1 "val1" key2 "val2" key3 "val3"
MGET key1 key2 key3
MSETNX key1 "val1" key2 "val2"

# With expiration
SETEX key 60 "value"      # expires in 60 seconds
PSETEX key 5000 "value"   # expires in 5000 milliseconds
TTL key                    # check time remaining (seconds)
PTTL key                   # check time remaining (milliseconds)

# String manipulation
APPEND key " more text"
STRLEN key
GETRANGE key 0 10
GETSET key "new value"

# Key management
RENAME oldkey newkey
FLUSHALL                   # ⚠️ DANGER: deletes everything
```
