# Redis CLI Commands Reference

## Connection & Testing

### PING - Test Server Connection

Tests if the Redis server is running and responsive.

```bash
PING
```

**Output:** `PONG`

### ECHO - Echo a Message

Returns the provided message, useful for testing connectivity.

```bash
ECHO "hello"
```

**Output:** `"hello"`

---

## String Operations

### SET - Create/Update a Key

Stores a value associated with a key.

**Syntax:** `SET key value`

```bash
SET name "Anurag Kumar"
```

**Output:** `OK`

### GET - Retrieve a Value

Retrieves the value stored at a key.

**Syntax:** `GET key`

```bash
GET name
```

**Output:** `"Anurag Kumar"`

---

## Key Management

### EXISTS - Check Key Existence

Checks if one or more keys exist in the database.

**Syntax:** `EXISTS key`

```bash
EXISTS name
```

**Output:** `(integer) 1` _(1 = exists, 0 = doesn't exist)_

### DEL - Delete a Key

Removes specified keys from the database.

**Syntax:** `DEL key`

```bash
DEL name
```

**Output:** `(integer) 1` _(number of keys deleted)_

```bash
EXISTS name
```

**Output:** `(integer) 0` _(key no longer exists)_

---

## Numeric Operations

### Working with Integers

```bash
SET count 10
```

**Output:** `OK`

```bash
GET count
```

**Output:** `"10"`

### INCR - Increment by 1

Increments the integer value of a key by 1.

```bash
INCR count
```

**Output:** `(integer) 11`

```bash
INCR count
```

**Output:** `(integer) 12`

### DECR - Decrement by 1

Decrements the integer value of a key by 1.

```bash
DECR count
```

**Output:** `(integer) 11`

---

## Command Line Execution

### Direct Command Execution

Execute Redis commands directly from the terminal without entering the Redis CLI.

**Syntax:** `redis-cli COMMAND [arguments]`

```bash
redis-cli ECHO "hello"
```

**Output:** `"hello"`

---

## Monitoring & Debugging

### MONITOR - Real-time Command Monitoring

Streams back every command processed by the Redis server in real-time.

```bash
MONITOR
```

**Output:** `OK`

After running this command, all subsequent Redis operations will be displayed as they occur.

---

## Namespacing with Keys

Using colons (`:`) to create logical namespaces for better key organization.

```bash
SET user:name "Anurag"
```

**Output:** `OK`

```bash
SET user:email "anuragxo.dev@gmail.com"
```

**Output:** `OK`

```bash
GET user:name
```

**Output:** `"Anurag"`

---

## Key Expiration

### EXPIRE - Set Key Expiration Time

Sets a timeout on a key, after which it will automatically be deleted.

**Syntax:** `EXPIRE key seconds`

```bash
EXPIRE user:name 100
```

**Output:** `(integer) 1` _(expiration set successfully)_

### TTL - Check Time to Live

Returns the remaining time to live of a key that has a timeout.

**Syntax:** `TTL key`

```bash
TTL user:name
```

**Output:** `(integer) 88` _(seconds remaining)_

**Possible TTL return values:**

- Positive integer: seconds until expiration
- `-1`: key exists but has no expiration
- `-2`: key does not exist

---

## Best Practices

- **Naming conventions:** Use descriptive names with namespaces (e.g., `user:123:email`)
- **Key expiration:** Always set expiration on temporary data to prevent memory bloat
- **Monitor sparingly:** Use `MONITOR` only for debugging as it impacts performance
- **Atomic operations:** Use `INCR`/`DECR` for counters to ensure thread-safety
