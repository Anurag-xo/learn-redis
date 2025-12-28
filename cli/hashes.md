# Redis Hash Commands

## Overview

Hashes in Redis are **maps between string fields and string values**, perfect for representing objects. They are memory-efficient and ideal for storing structured data like user profiles, product details, or configuration settings.

---

## Hash Characteristics

- **Field-value pairs** - Similar to objects/dictionaries in programming languages
- **Flat structure** - No nested hashes (single level only)
- **Memory efficient** - Optimized encoding for small hashes
- **Max fields** - Up to 2^32 - 1 field-value pairs per hash
- **String values** - All values are strings (can represent numbers)

---

## Basic Hash Commands

### HSET - Set Field Value

Sets the value of a field in a hash. Creates the hash if it doesn't exist.

**Syntax:** `HSET key field value [field value ...]`

```bash
HSET user:1000 name "Anurag Kumar"
```

**Output:** `(integer) 1` _(number of fields added)_

**Set multiple fields at once:**

```bash
HSET user:1000 email "anurag@example.com" age "25" city "Indore"
```

**Output:** `(integer) 3`

**Update existing field:**

```bash
HSET user:1000 age "26"
```

**Output:** `(integer) 0` _(field existed, so 0 new fields added)_

### HGET - Get Field Value

Retrieves the value of a specific field.

**Syntax:** `HGET key field`

```bash
HGET user:1000 name
```

**Output:** `"Anurag Kumar"`

```bash
HGET user:1000 email
```

**Output:** `"anurag@example.com"`

**Non-existent field:**

```bash
HGET user:1000 phone
```

**Output:** `(nil)`

### HSETNX - Set Field Only if Not Exists

Sets field value only if the field doesn't already exist.

**Syntax:** `HSETNX key field value`

```bash
HSETNX user:1000 country "India"
```

**Output:** `(integer) 1` _(field was set)_

```bash
HSETNX user:1000 country "USA"
```

**Output:** `(integer) 0` _(field already exists, not modified)_

```bash
HGET user:1000 country
```

**Output:** `"India"` _(original value preserved)_

**Use case:** Setting default values, preventing overwrites

### HMSET - Set Multiple Fields (Deprecated)

Sets multiple field-value pairs. **Note:** Deprecated since Redis 4.0.0, use `HSET` instead.

**Syntax:** `HMSET key field value [field value ...]`

```bash
HMSET user:2000 name "Bob" email "bob@example.com" age "30"
```

**Output:** `OK`

**Modern alternative:**

```bash
HSET user:2000 name "Bob" email "bob@example.com" age "30"
```

### HMGET - Get Multiple Field Values

Retrieves values of multiple fields in a single operation.

**Syntax:** `HMGET key field [field ...]`

```bash
HMGET user:1000 name email city
```

**Output:**

```
1) "Anurag Kumar"
2) "anurag@example.com"
3) "Indore"
```

**With non-existent fields:**

```bash
HMGET user:1000 name phone country
```

**Output:**

```
1) "Anurag Kumar"
2) (nil)
3) "India"
```

**Use case:** Efficiently fetching multiple object properties

---

## Retrieving Hash Data

### HGETALL - Get All Fields and Values

Returns all field-value pairs in the hash.

**Syntax:** `HGETALL key`

```bash
HSET product:500 name "Laptop" price "999" stock "50" brand "TechCorp"
HGETALL product:500
```

**Output:**

```
1) "name"
2) "Laptop"
3) "price"
4) "999"
5) "stock"
6) "50"
7) "brand"
8) "TechCorp"
```

**⚠️ Warning:** Use cautiously on large hashes in production (O(N) operation)

### HKEYS - Get All Field Names

Returns all field names in the hash.

**Syntax:** `HKEYS key`

```bash
HKEYS product:500
```

**Output:**

```
1) "name"
2) "price"
3) "stock"
4) "brand"
```

**Use case:** Discovering available fields, schema inspection

### HVALS - Get All Values

Returns all values in the hash.

**Syntax:** `HVALS key`

```bash
HVALS product:500
```

**Output:**

```
1) "Laptop"
2) "999"
3) "50"
4) "TechCorp"
```

### HLEN - Get Number of Fields

Returns the number of fields in the hash.

**Syntax:** `HLEN key`

```bash
HLEN product:500
```

**Output:** `(integer) 4`

**Non-existent hash:**

```bash
HLEN nonexistent
```

**Output:** `(integer) 0`

### HEXISTS - Check if Field Exists

Checks whether a field exists in the hash.

**Syntax:** `HEXISTS key field`

```bash
HEXISTS user:1000 email
```

**Output:** `(integer) 1` _(field exists)_

```bash
HEXISTS user:1000 phone
```

**Output:** `(integer) 0` _(field doesn't exist)_

**Use case:** Conditional logic, validation before operations

---

## Deleting Hash Data

### HDEL - Delete Fields

Removes one or more fields from the hash.

**Syntax:** `HDEL key field [field ...]`

```bash
HSET session:abc token "xyz123" user "alice" expires "3600"
HDEL session:abc expires
```

**Output:** `(integer) 1` _(number of fields removed)_

**Delete multiple fields:**

```bash
HDEL session:abc token user
```

**Output:** `(integer) 2`

```bash
HGETALL session:abc
```

**Output:** `(empty array)`

**Delete non-existent field:**

```bash
HDEL session:abc nonexistent
```

**Output:** `(integer) 0`

---

## Numeric Operations on Hash Fields

### HINCRBY - Increment Field by Integer

Increments the integer value of a field by the specified amount.

**Syntax:** `HINCRBY key field increment`

```bash
HSET stats:daily views 100 likes 50
HINCRBY stats:daily views 1
```

**Output:** `(integer) 101`

```bash
HINCRBY stats:daily views 10
```

**Output:** `(integer) 111`

**Decrement (negative increment):**

```bash
HINCRBY stats:daily likes -5
```

**Output:** `(integer) 45`

**Field doesn't exist (starts at 0):**

```bash
HINCRBY stats:daily shares 1
```

**Output:** `(integer) 1`

**Error handling:**

```bash
HSET user:1000 name "Alice"
HINCRBY user:1000 name 1
```

**Output:** `(error) ERR hash value is not an integer`

### HINCRBYFLOAT - Increment Field by Float

Increments the float value of a field by the specified amount.

**Syntax:** `HINCRBYFLOAT key field increment`

```bash
HSET account:1234 balance 1000.50
HINCRBYFLOAT account:1234 balance 250.75
```

**Output:** `"1251.25"`

**Decrement:**

```bash
HINCRBYFLOAT account:1234 balance -100.25
```

**Output:** `"1151"`

**Scientific notation:**

```bash
HINCRBYFLOAT metrics:cpu usage 1.5e-2
```

**Output:** `"0.015"`

**Use case:** Financial calculations, precise measurements

---

## Advanced Hash Operations

### HSTRLEN - Get Field Value Length

Returns the string length of the value stored at field.

**Syntax:** `HSTRLEN key field`

```bash
HSET user:1000 bio "Redis enthusiast and developer"
HSTRLEN user:1000 bio
```

**Output:** `(integer) 31`

```bash
HSTRLEN user:1000 name
```

**Output:** `(integer) 13` _("Anurag Kumar")_

**Non-existent field:**

```bash
HSTRLEN user:1000 phone
```

**Output:** `(integer) 0`

### HSCAN - Incrementally Iterate Hash Fields

Iterates over fields in a hash without blocking the server.

**Syntax:** `HSCAN key cursor [MATCH pattern] [COUNT count]`

```bash
# Add many fields
HSET inventory:store item1 100 item2 200 item3 150 item4 300 item5 50

# Start scan (cursor 0)
HSCAN inventory:store 0
```

**Output:**

```
1) "0"  (next cursor, 0 means complete)
2) 1) "item1"
   2) "100"
   3) "item2"
   4) "200"
   5) "item3"
   6) "150"
   7) "item4"
   8) "300"
   9) "item5"
   10) "50"
```

**With pattern matching:**

```bash
HSET products prod:1 "Laptop" prod:2 "Mouse" prod:3 "Keyboard" cat:1 "Electronics"
HSCAN products 0 MATCH prod:*
```

**Output:**

```
1) "0"
2) 1) "prod:1"
   2) "Laptop"
   3) "prod:2"
   4) "Mouse"
   5) "prod:3"
   6) "Keyboard"
```

**With count hint:**

```bash
HSCAN products 0 COUNT 2
```

**Output:** Returns approximately 2 field-value pairs per iteration

**Use case:** Safe iteration over large hashes, pattern-based filtering

### HRANDFIELD - Get Random Field(s)

Returns random field(s) from the hash.

**Syntax:** `HRANDFIELD key [count [WITHVALUES]]`

```bash
HSET lottery user1 "Alice" user2 "Bob" user3 "Charlie" user4 "Dave"
HRANDFIELD lottery
```

**Output:** `"user3"` _(random field name)_

**Get multiple random fields:**

```bash
HRANDFIELD lottery 2
```

**Output:**

```
1) "user1"
2) "user4"
```

**Get fields with values:**

```bash
HRANDFIELD lottery 2 WITHVALUES
```

**Output:**

```
1) "user2"
2) "Bob"
3) "user3"
4) "Charlie"
```

**Negative count (allow repetitions):**

```bash
HRANDFIELD lottery -5 WITHVALUES
```

**Output:** May return same field multiple times

**Use case:** Random selection, sampling, lottery systems

---

## Practical Examples

### Example 1: User Profile Management

```bash
# Create user profile
HSET user:1001 username "john_doe" email "john@example.com" name "John Doe" joined "2025-01-15" status "active"

# Get specific fields
HMGET user:1001 username email status
# Output:
# 1) "john_doe"
# 2) "john@example.com"
# 3) "active"

# Update profile
HSET user:1001 status "premium" last_login "2025-12-28"

# Check if field exists
HEXISTS user:1001 phone
# Output: (integer) 0

# Add optional field
HSETNX user:1001 phone "+91-9876543210"

# Get complete profile
HGETALL user:1001
```

### Example 2: E-commerce Product Catalog

```bash
# Add product
HSET product:2001 name "Wireless Mouse" price "29.99" stock "150" category "Electronics" brand "TechGear" rating "4.5"

# Update stock after sale
HINCRBY product:2001 stock -1
# Output: (integer) 149

# Update price
HSET product:2001 price "24.99"

# Check availability
HGET product:2001 stock
# Output: "149"

# Get product details for display
HMGET product:2001 name price rating
# Output:
# 1) "Wireless Mouse"
# 2) "24.99"
# 3) "4.5"
```

### Example 3: Session Management

```bash
# Create session
HSET session:xyz789 user_id "1001" username "john_doe" ip "192.168.1.100" created "1735372800"

# Update last activity
HSET session:xyz789 last_activity "1735383600"

# Increment page views
HINCRBY session:xyz789 page_views 1
# Output: (integer) 1

# Check session validity
HEXISTS session:xyz789 user_id
# Output: (integer) 1

# Get session info
HGETALL session:xyz789

# Delete session on logout
DEL session:xyz789
```

### Example 4: Real-time Analytics Counter

```bash
# Initialize daily metrics
HSET metrics:2025-12-28 visits 0 sales 0 revenue 0.0 signups 0

# Track events
HINCRBY metrics:2025-12-28 visits 1
HINCRBY metrics:2025-12-28 visits 1
HINCRBY metrics:2025-12-28 sales 1
HINCRBYFLOAT metrics:2025-12-28 revenue 99.99
HINCRBY metrics:2025-12-28 signups 1

# Get metrics snapshot
HGETALL metrics:2025-12-28
# Output:
# 1) "visits"
# 2) "2"
# 3) "sales"
# 4) "1"
# 5) "revenue"
# 6) "99.99"
# 7) "signups"
# 8) "1"

# Get specific metrics
HMGET metrics:2025-12-28 visits revenue
```

### Example 5: Configuration Management

```bash
# Application config
HSET config:app debug "false" max_connections "100" timeout "30" cache_enabled "true" log_level "info"

# Get single config value
HGET config:app debug
# Output: "false"

# Update config
HSET config:app debug "true" log_level "debug"

# Get all config keys
HKEYS config:app
# Output:
# 1) "debug"
# 2) "max_connections"
# 3) "timeout"
# 4) "cache_enabled"
# 5) "log_level"

# Check if feature enabled
HEXISTS config:app cache_enabled
# Output: (integer) 1
```

### Example 6: Shopping Cart

```bash
# Add items to cart (quantity as value)
HSET cart:user:5000 product:101 "2" product:205 "1" product:310 "3"

# Update quantity
HINCRBY cart:user:5000 product:101 1
# Output: (integer) 3

# Remove item
HDEL cart:user:5000 product:205

# Get cart items
HGETALL cart:user:5000
# Output:
# 1) "product:101"
# 2) "3"
# 3) "product:310"
# 4) "3"

# Count items in cart
HLEN cart:user:5000
# Output: (integer) 2

# Clear cart
DEL cart:user:5000
```

### Example 7: API Rate Limiting

```bash
# Track API usage per endpoint
HSET rate_limit:user:1001 /api/users 10 /api/products 25 /api/orders 5

# Increment request count
HINCRBY rate_limit:user:1001 /api/users 1
# Output: (integer) 11

# Check if limit exceeded
HGET rate_limit:user:1001 /api/users
# Output: "11"

# Reset specific endpoint
HSET rate_limit:user:1001 /api/users 0

# Get all endpoint usage
HGETALL rate_limit:user:1001
```

### Example 8: Feature Flags

```bash
# Set feature flags
HSET features:org:acme new_ui "enabled" beta_api "disabled" advanced_search "enabled" dark_mode "enabled"

# Check if feature enabled
HGET features:org:acme new_ui
# Output: "enabled"

# Toggle feature
HSET features:org:acme beta_api "enabled"

# Get all enabled features (requires client-side filtering)
HGETALL features:org:acme

# Get multiple feature states
HMGET features:org:acme new_ui beta_api dark_mode
# Output:
# 1) "enabled"
# 2) "enabled"
# 3) "enabled"
```

### Example 9: Leaderboard with Stats

```bash
# Player stats
HSET player:alice score 1500 level 25 wins 120 losses 80

# Update after game
HINCRBY player:alice score 50
HINCRBY player:alice wins 1

# Get player profile
HGETALL player:alice
# Output:
# 1) "score"
# 2) "1550"
# 3) "level"
# 4) "25"
# 5) "wins"
# 6) "121"
# 7) "losses"
# 8) "80"

# Calculate win rate (client-side)
HMGET player:alice wins losses
```

### Example 10: Caching Database Records

```bash
# Cache user record from database
HSET cache:user:999 id "999" username "sarah" email "sarah@example.com" role "admin" created "2024-01-10"

# Set expiration on hash (entire object)
EXPIRE cache:user:999 3600

# Check cache
HGET cache:user:999 username
# Output: "sarah"

# Update cache field
HSET cache:user:999 last_seen "2025-12-28"

# Check TTL
TTL cache:user:999
# Output: (integer) 3542
```

---

## Command Comparison Table

| Command        | Purpose                   | Creates Hash | Returns                    |
| -------------- | ------------------------- | ------------ | -------------------------- |
| `HSET`         | Set field(s)              | Yes          | Count of new fields        |
| `HGET`         | Get field value           | No           | String value or nil        |
| `HSETNX`       | Set if not exists         | Yes          | 1 (set) or 0 (not set)     |
| `HMSET`        | Set multiple (deprecated) | Yes          | OK                         |
| `HMGET`        | Get multiple fields       | No           | Array of values            |
| `HGETALL`      | Get all fields & values   | No           | Array (field, value pairs) |
| `HKEYS`        | Get all field names       | No           | Array of field names       |
| `HVALS`        | Get all values            | No           | Array of values            |
| `HLEN`         | Get field count           | No           | Integer count              |
| `HEXISTS`      | Check field exists        | No           | 1 (exists) or 0 (not)      |
| `HDEL`         | Delete field(s)           | No           | Count of deleted fields    |
| `HINCRBY`      | Increment by integer      | Yes          | New value                  |
| `HINCRBYFLOAT` | Increment by float        | Yes          | New value (string)         |
| `HSTRLEN`      | Get value length          | No           | Integer length             |
| `HSCAN`        | Iterate fields            | No           | Cursor + field-value pairs |
| `HRANDFIELD`   | Get random field(s)       | No           | Field name(s) [+ values]   |

---

## Performance Characteristics

| Operation                | Time Complexity | Notes                          |
| ------------------------ | --------------- | ------------------------------ |
| `HSET`                   | O(1)            | Per field                      |
| `HGET`                   | O(1)            | Single field access            |
| `HMSET`/`HMGET`          | O(N)            | N = number of fields           |
| `HGETALL`                | O(N)            | N = total fields in hash       |
| `HKEYS`/`HVALS`          | O(N)            | N = total fields in hash       |
| `HLEN`                   | O(1)            | Cached value                   |
| `HEXISTS`                | O(1)            | Direct field lookup            |
| `HDEL`                   | O(N)            | N = number of fields to delete |
| `HINCRBY`/`HINCRBYFLOAT` | O(1)            | Single field operation         |
| `HSTRLEN`                | O(1)            | Direct field access            |
| `HSCAN`                  | O(1)            | Per iteration call             |
| `HRANDFIELD`             | O(N)            | N = count parameter            |

---

## Best Practices

✅ **Do:**

- Use hashes for objects with multiple attributes
- Prefer `HMGET` over multiple `HGET` calls
- Use `HINCRBY` for atomic counter updates
- Use `HSETNX` to prevent accidental overwrites
- Set expiration on entire hash keys when appropriate
- Use `HSCAN` for iterating large hashes

❌ **Don't:**

- Nest hashes (use separate keys with namespacing instead)
- Use `HGETALL` frequently on large hashes in production
- Store binary data in hash values (use strings instead)
- Use hashes for simple key-value pairs (strings are more efficient)
- Forget that all values are strings (convert when needed)
- Use hashes for sorted data (use sorted sets instead)

---

## Memory Optimization

### Small Hash Encoding

Redis uses special memory-efficient encoding for small hashes:

```bash
# Configure thresholds (in redis.conf or CONFIG SET)
CONFIG SET hash-max-ziplist-entries 512
CONFIG SET hash-max-ziplist-value 64
```

**When to use hashes for memory efficiency:**

- Storing many small objects
- Each object has < 512 fields
- Each value is < 64 bytes

**Example:**

```bash
# More efficient than 100 separate keys
HSET user:sessions session:1 "data1" session:2 "data2" ... session:100 "data100"
```

---

## Common Patterns

### Object Storage

```bash
HSET object:id field1 value1 field2 value2 ...
HGETALL object:id
```

### Counters with Metadata

```bash
HSET stats:page visits 1000 unique 750 bounce_rate 25.5
HINCRBY stats:page visits 1
```

### Configuration Dictionary

```bash
HSET config key1 value1 key2 value2
HGET config key1
```

### Temporary Data with Expiration

```bash
HSET temp:data field value
EXPIRE temp:data 300
```

---

## Quick Reference

```bash
# Set operations
HSET key field value [field value ...]    # Set field(s)
HSETNX key field value                    # Set if not exists
HMSET key field value [field value ...]   # Deprecated, use HSET

# Get operations
HGET key field                            # Get single field
HMGET key field [field ...]               # Get multiple fields
HGETALL key                               # Get all fields & values
HKEYS key                                 # Get all field names
HVALS key                                 # Get all values

# Check & count
HEXISTS key field                         # Check field exists
HLEN key                                  # Count fields
HSTRLEN key field                         # Get value length

# Delete
HDEL key field [field ...]                # Delete field(s)

# Numeric operations
HINCRBY key field increment               # Increment by integer
HINCRBYFLOAT key field increment          # Increment by float

# Advanced
HSCAN key cursor [MATCH pattern] [COUNT count]  # Iterate fields
HRANDFIELD key [count [WITHVALUES]]            # Random field(s)
```

---

## Hash vs Other Data Types

| Use Case                          | Hash    | String  | List    | Set     |
| --------------------------------- | ------- | ------- | ------- | ------- |
| Object with fields                | ✅ Best | ❌      | ❌      | ❌      |
| Simple key-value                  | ❌      | ✅ Best | ❌      | ❌      |
| Ordered collection                | ❌      | ❌      | ✅ Best | ❌      |
| Unique values                     | ❌      | ❌      | ❌      | ✅ Best |
| Counters                          | ✅ Good | ✅ Good | ❌      | ❌      |
| Memory efficiency (small objects) | ✅ Best | ❌      | ❌      | ❌      |
