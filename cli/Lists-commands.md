# Redis List Commands

## Overview

Lists in Redis are **ordered collections of strings**, sorted by insertion order. They are implemented as **linked lists**, making them extremely efficient for operations at the head and tail, even with millions of elements.

---

## List Characteristics

- **Ordered** - Elements maintain insertion order
- **Indexed** - Access elements by position (0-based indexing)
- **Duplicates allowed** - Same value can appear multiple times
- **Max length** - Up to 2^32 - 1 elements (over 4 billion)
- **Performance** - O(1) for head/tail operations, O(N) for middle operations

---

## Basic List Commands

### LPUSH - Push to Head (Left)

Inserts one or more values at the head (left/beginning) of the list.

**Syntax:** `LPUSH key value [value ...]`

```bash
LPUSH tasks "task1"
```

**Output:** `(integer) 1` _(list length after operation)_

```bash
LPUSH tasks "task2" "task3"
```

**Output:** `(integer) 3`

**List order:** `["task3", "task2", "task1"]`

**Use case:** Stack (LIFO), recent items first

### RPUSH - Push to Tail (Right)

Inserts one or more values at the tail (right/end) of the list.

**Syntax:** `RPUSH key value [value ...]`

```bash
RPUSH queue "job1"
```

**Output:** `(integer) 1`

```bash
RPUSH queue "job2" "job3"
```

**Output:** `(integer) 3`

**List order:** `["job1", "job2", "job3"]`

**Use case:** Queue (FIFO), append operations

### LPUSHX - Push to Head (Only if List Exists)

Inserts values at the head only if the list already exists.

**Syntax:** `LPUSHX key value [value ...]`

```bash
LPUSHX nonexistent "value"
```

**Output:** `(integer) 0` _(list doesn't exist, nothing added)_

```bash
LPUSH mylist "first"
LPUSHX mylist "second"
```

**Output:** `(integer) 2` _(list exists, value added)_

### RPUSHX - Push to Tail (Only if List Exists)

Inserts values at the tail only if the list already exists.

**Syntax:** `RPUSHX key value [value ...]`

```bash
RPUSH myqueue "item1"
RPUSHX myqueue "item2"
```

**Output:** `(integer) 2`

```bash
RPUSHX newqueue "item"
```

**Output:** `(integer) 0` _(list doesn't exist)_

---

## Retrieving Elements

### LRANGE - Get Range of Elements

Returns elements from a list within the specified range.

**Syntax:** `LRANGE key start stop`

```bash
RPUSH numbers 1 2 3 4 5 6 7 8 9 10
LRANGE numbers 0 4
```

**Output:**

```
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

**Get all elements:**

```bash
LRANGE numbers 0 -1
```

**Output:** All elements from index 0 to end

**Negative indices** (count from end):

```bash
LRANGE numbers -3 -1
```

**Output:**

```
1) "8"
2) "9"
3) "10"
```

**Use case:** Pagination, retrieving recent items

### LINDEX - Get Element by Index

Returns the element at the specified index.

**Syntax:** `LINDEX key index`

```bash
RPUSH fruits "apple" "banana" "cherry" "date"
LINDEX fruits 0
```

**Output:** `"apple"`

```bash
LINDEX fruits 2
```

**Output:** `"cherry"`

**Negative indices:**

```bash
LINDEX fruits -1
```

**Output:** `"date"` _(last element)_

```bash
LINDEX fruits -2
```

**Output:** `"cherry"`

If index out of range:

```bash
LINDEX fruits 100
```

**Output:** `(nil)`

### LLEN - Get List Length

Returns the number of elements in the list.

**Syntax:** `LLEN key`

```bash
RPUSH items "a" "b" "c" "d" "e"
LLEN items
```

**Output:** `(integer) 5`

For non-existent lists:

```bash
LLEN nonexistent
```

**Output:** `(integer) 0`

---

## Removing Elements

### LPOP - Remove and Return from Head

Removes and returns the first element(s) from the list.

**Syntax:** `LPOP key [count]`

```bash
RPUSH stack "bottom" "middle" "top"
LPOP stack
```

**Output:** `"bottom"`

**Pop multiple elements:**

```bash
LPOP stack 2
```

**Output:**

```
1) "middle"
2) "top"
```

**Current list:** `[]` _(empty)_

### RPOP - Remove and Return from Tail

Removes and returns the last element(s) from the list.

**Syntax:** `RPOP key [count]`

```bash
RPUSH queue "first" "second" "third" "fourth"
RPOP queue
```

**Output:** `"fourth"`

**Pop multiple elements:**

```bash
RPOP queue 2
```

**Output:**

```
1) "third"
2) "second"
```

**Current list:** `["first"]`

### LREM - Remove Elements by Value

Removes the first count occurrences of elements equal to value.

**Syntax:** `LREM key count value`

**Count behavior:**

- `count > 0` - Remove from head to tail
- `count < 0` - Remove from tail to head
- `count = 0` - Remove all occurrences

```bash
RPUSH colors "red" "blue" "red" "green" "red" "yellow"
LREM colors 2 "red"
```

**Output:** `(integer) 2` _(number of elements removed)_

**Current list:** `["blue", "green", "red", "yellow"]`

**Remove from tail:**

```bash
RPUSH nums 1 2 3 2 4 2 5
LREM nums -2 2
```

**Output:** `(integer) 2`

**Current list:** `[1, 3, 2, 4, 5]` _(removed last two 2's)_

**Remove all occurrences:**

```bash
RPUSH letters "a" "b" "a" "c" "a"
LREM letters 0 "a"
```

**Output:** `(integer) 3`

**Current list:** `["b", "c"]`

### LTRIM - Trim List to Range

Keeps only elements within the specified range, removing all others.

**Syntax:** `LTRIM key start stop`

```bash
RPUSH logs "log1" "log2" "log3" "log4" "log5" "log6"
LTRIM logs 0 2
```

**Output:** `OK`

**Current list:** `["log1", "log2", "log3"]`

**Keep last 3 elements:**

```bash
RPUSH recent 1 2 3 4 5 6 7 8 9 10
LTRIM recent -3 -1
```

**Output:** `OK`

**Current list:** `["8", "9", "10"]`

**Use case:** Keeping only recent N items (capped lists)

---

## Modifying Elements

### LSET - Set Element at Index

Sets the value of an element at a specific index.

**Syntax:** `LSET key index value`

```bash
RPUSH users "Alice" "Bob" "Charlie"
LSET users 1 "Robert"
```

**Output:** `OK`

**Current list:** `["Alice", "Robert", "Charlie"]`

**Using negative indices:**

```bash
LSET users -1 "Charles"
```

**Output:** `OK`

**Current list:** `["Alice", "Robert", "Charles"]`

**Error handling:**

```bash
LSET users 10 "Dave"
```

**Output:** `(error) ERR index out of range`

```bash
LSET nonexistent 0 "value"
```

**Output:** `(error) ERR no such key`

### LINSERT - Insert Before or After Element

Inserts a value before or after a pivot element.

**Syntax:** `LINSERT key BEFORE|AFTER pivot value`

```bash
RPUSH priorities "low" "high"
LINSERT priorities BEFORE "high" "medium"
```

**Output:** `(integer) 3` _(new list length)_

**Current list:** `["low", "medium", "high"]`

**Insert after:**

```bash
LINSERT priorities AFTER "medium" "medium-high"
```

**Output:** `(integer) 4`

**Current list:** `["low", "medium", "medium-high", "high"]`

**If pivot not found:**

```bash
LINSERT priorities BEFORE "critical" "urgent"
```

**Output:** `(integer) -1`

---

## Blocking Operations

### BLPOP - Blocking Left Pop

Blocks until an element is available, then removes and returns it from the head.

**Syntax:** `BLPOP key [key ...] timeout`

```bash
# In terminal 1
BLPOP notifications 0
# Blocks indefinitely (timeout 0) waiting for data
```

```bash
# In terminal 2
LPUSH notifications "New message"
```

**Terminal 1 output:**

```
1) "notifications"
2) "New message"
```

**With timeout:**

```bash
BLPOP tasks 5
# Waits up to 5 seconds
```

If no data arrives:
**Output:** `(nil)` _(after 5 seconds)_

**Multiple keys:**

```bash
BLPOP queue1 queue2 queue3 30
# Checks keys in order, blocks for 30 seconds
```

**Use case:** Task queues, message consumers

### BRPOP - Blocking Right Pop

Blocks until an element is available, then removes and returns it from the tail.

**Syntax:** `BRPOP key [key ...] timeout`

```bash
BRPOP jobs 10
# Blocks for up to 10 seconds waiting for a job
```

### BRPOPLPUSH - Blocking Pop and Push

Atomically pops from one list's tail and pushes to another's head, with blocking.

**Syntax:** `BRPOPLPUSH source destination timeout`

```bash
# Reliable queue pattern
BRPOPLPUSH pending processing 0
# Moves job from pending to processing, blocks until available
```

**Use case:** Reliable task processing with backup

### BLMOVE - Blocking Move Between Lists

Pops from source and pushes to destination, with blocking and directional control.

**Syntax:** `BLMOVE source destination LEFT|RIGHT LEFT|RIGHT timeout`

```bash
BLMOVE queue1 queue2 RIGHT LEFT 5
# Pop from right of queue1, push to left of queue2, wait 5 seconds
```

---

## Advanced List Operations

### RPOPLPUSH - Atomic Pop and Push

Atomically removes element from tail of source and pushes to head of destination.

**Syntax:** `RPOPLPUSH source destination`

```bash
RPUSH todo "task1" "task2" "task3"
RPOPLPUSH todo doing
```

**Output:** `"task3"`

**Lists after:**

- `todo`: `["task1", "task2"]`
- `doing`: `["task3"]`

**Circular list pattern (same source and destination):**

```bash
RPUSH playlist "song1" "song2" "song3"
RPOPLPUSH playlist playlist
```

**Output:** `"song3"`

**Current list:** `["song3", "song1", "song2"]` _(rotated)_

### LMOVE - Move Element Between Lists

Moves element from source to destination with directional control.

**Syntax:** `LMOVE source destination LEFT|RIGHT LEFT|RIGHT`

```bash
RPUSH source "a" "b" "c"
LMOVE source dest RIGHT LEFT
```

**Output:** `"c"`

**Lists after:**

- `source`: `["a", "b"]`
- `dest`: `["c"]`

**All direction combinations:**

```bash
LMOVE list1 list2 LEFT RIGHT   # Pop from head, push to tail
LMOVE list1 list2 LEFT LEFT    # Pop from head, push to head
LMOVE list1 list2 RIGHT RIGHT  # Pop from tail, push to tail
LMOVE list1 list2 RIGHT LEFT   # Pop from tail, push to head
```

### LPOS - Find Position of Element

Returns the index of matching elements in the list.

**Syntax:** `LPOS key element [RANK rank] [COUNT num] [MAXLEN len]`

```bash
RPUSH items "apple" "banana" "cherry" "banana" "date"
LPOS items "banana"
```

**Output:** `(integer) 1` _(first occurrence)_

**Find second occurrence:**

```bash
LPOS items "banana" RANK 2
```

**Output:** `(integer) 3`

**Find all occurrences:**

```bash
LPOS items "banana" COUNT 0
```

**Output:**

```
1) (integer) 1
2) (integer) 3
```

**Limit search:**

```bash
LPOS items "banana" COUNT 1 MAXLEN 3
# Search only first 3 elements
```

---

## Practical Examples

### Example 1: Task Queue (FIFO)

```bash
# Producer adds tasks
RPUSH task:queue "send_email:user@example.com"
RPUSH task:queue "generate_report:monthly"
RPUSH task:queue "cleanup_temp_files"

# Consumer processes tasks
LPOP task:queue
# Output: "send_email:user@example.com"

# Check remaining tasks
LLEN task:queue
# Output: (integer) 2

# View all pending tasks
LRANGE task:queue 0 -1
```

### Example 2: Activity Feed (Recent Items)

```bash
# Add new activities
LPUSH user:123:feed "commented on post"
LPUSH user:123:feed "liked photo"
LPUSH user:123:feed "shared article"

# Keep only last 100 activities
LTRIM user:123:feed 0 99

# Get recent 10 activities
LRANGE user:123:feed 0 9
```

### Example 3: Undo/Redo Stack

```bash
# User actions
RPUSH user:456:history "typed 'Hello'"
RPUSH user:456:history "typed ' World'"
RPUSH user:456:history "deleted 'World'"

# Undo (move to redo stack)
RPOPLPUSH user:456:history user:456:redo
# Output: "deleted 'World'"

# Redo (move back)
RPOPLPUSH user:456:redo user:456:history
# Output: "deleted 'World'"
```

### Example 4: Reliable Job Processing

```bash
# Job arrives
RPUSH jobs:pending "process_payment:12345"

# Worker claims job (atomic move)
RPOPLPUSH jobs:pending jobs:processing
# Output: "process_payment:12345"

# After successful processing
LREM jobs:processing 1 "process_payment:12345"

# If worker crashes, job stays in processing for recovery
LRANGE jobs:processing 0 -1
```

### Example 5: Leaderboard (Top Scores)

```bash
# Add scores with player names
LPUSH leaderboard "Alice:1500"
LPUSH leaderboard "Bob:1800"
LPUSH leaderboard "Charlie:1200"

# Get top 3
LRANGE leaderboard 0 2
```

### Example 6: Removing Duplicates

```bash
RPUSH tags "redis" "database" "redis" "cache" "redis" "nosql"

# Remove all "redis" duplicates
LREM tags 0 "redis"
# Output: (integer) 3

# Re-add once
LPUSH tags "redis"

# Final list
LRANGE tags 0 -1
# Output: ["redis", "database", "cache", "nosql"]
```

### Example 7: Carousel/Rotation

```bash
RPUSH playlist "song1" "song2" "song3" "song4"

# Play next and rotate
RPOPLPUSH playlist playlist
# Output: "song4"

# List is now: ["song4", "song1", "song2", "song3"]

# Continue rotating
RPOPLPUSH playlist playlist
# Output: "song3"
```

### Example 8: Message Queue with Priority

```bash
# High priority queue
LPUSH messages:high "urgent_notification"

# Normal priority queue
LPUSH messages:normal "regular_update"

# Consumer checks high priority first
LPOP messages:high
# Output: "urgent_notification"

# If empty, check normal
LPOP messages:normal
# Output: "regular_update"
```

---

## Command Comparison Table

| Command      | Position  | Blocking | Creates if Missing | Returns           |
| ------------ | --------- | -------- | ------------------ | ----------------- |
| `LPUSH`      | Head      | No       | Yes                | List length       |
| `RPUSH`      | Tail      | No       | Yes                | List length       |
| `LPUSHX`     | Head      | No       | No                 | List length       |
| `RPUSHX`     | Tail      | No       | No                 | List length       |
| `LPOP`       | Head      | No       | N/A                | Element(s)        |
| `RPOP`       | Tail      | No       | N/A                | Element(s)        |
| `BLPOP`      | Head      | Yes      | N/A                | Key + Element     |
| `BRPOP`      | Tail      | Yes      | N/A                | Key + Element     |
| `LRANGE`     | Range     | No       | N/A                | Array of elements |
| `LINDEX`     | Specific  | No       | N/A                | Single element    |
| `LLEN`       | N/A       | No       | N/A                | Integer count     |
| `LSET`       | Specific  | No       | No                 | OK                |
| `LINSERT`    | Relative  | No       | No                 | List length or -1 |
| `LREM`       | By value  | No       | N/A                | Count removed     |
| `LTRIM`      | Range     | No       | N/A                | OK                |
| `RPOPLPUSH`  | Tail→Head | No       | Yes (dest)         | Element moved     |
| `BRPOPLPUSH` | Tail→Head | Yes      | Yes (dest)         | Element moved     |

---

## Performance Characteristics

| Operation       | Time Complexity | Notes                                   |
| --------------- | --------------- | --------------------------------------- |
| `LPUSH`/`RPUSH` | O(1)            | Per element                             |
| `LPOP`/`RPOP`   | O(1)            | Per element                             |
| `LINDEX`        | O(N)            | N = index distance from head/tail       |
| `LSET`          | O(N)            | N = index distance from head/tail       |
| `LRANGE`        | O(S+N)          | S = start offset, N = elements returned |
| `LLEN`          | O(1)            | Cached value                            |
| `LREM`          | O(N+M)          | N = list length, M = count              |
| `LTRIM`         | O(N)            | N = elements removed                    |
| `LINSERT`       | O(N)            | N = position of pivot                   |
| `RPOPLPUSH`     | O(1)            | Atomic operation                        |

---

## Best Practices

✅ **Do:**

- Use `LPUSH`/`RPOP` for queues (FIFO)
- Use `LPUSH`/`LPOP` for stacks (LIFO)
- Use `LTRIM` to implement capped collections
- Use `RPOPLPUSH` for reliable queue processing
- Use `BLPOP`/`BRPOP` for worker pools
- Access elements by head/tail for O(1) performance

❌ **Don't:**

- Use lists as sets (no uniqueness guarantee)
- Frequently access middle elements (slow)
- Store huge lists without pagination
- Use `LRANGE 0 -1` on very large lists in production
- Forget to handle `(nil)` returns from pop operations

---

## Common Patterns

### Producer-Consumer Queue

```bash
# Producer
RPUSH jobs "job1" "job2" "job3"

# Consumer (blocking wait)
BLPOP jobs 30
```

### Capped Collection (Latest N Items)

```bash
LPUSH recent:logins "user:123"
LTRIM recent:logins 0 999  # Keep last 1000
```

### Reliable Queue (with backup)

```bash
# Move to processing
RPOPLPUSH pending processing

# On success, remove
LREM processing 1 "job"

# On failure, move back
RPOPLPUSH processing pending
```

### Circular Buffer

```bash
RPOPLPUSH buffer buffer  # Rotate elements
```

---

## Quick Reference

```bash
# Add elements
LPUSH key value [value ...]       # Add to head
RPUSH key value [value ...]       # Add to tail
LPUSHX key value                  # Add to head if exists
RPUSHX key value                  # Add to tail if exists

# Remove elements
LPOP key [count]                  # Remove from head
RPOP key [count]                  # Remove from tail
LREM key count value              # Remove by value
LTRIM key start stop              # Keep only range

# Retrieve elements
LRANGE key start stop             # Get range
LINDEX key index                  # Get by index
LLEN key                          # Get length
LPOS key element                  # Find position

# Modify elements
LSET key index value              # Set by index
LINSERT key BEFORE|AFTER pivot value  # Insert relative

# Blocking operations
BLPOP key [key ...] timeout       # Blocking pop from head
BRPOP key [key ...] timeout       # Blocking pop from tail

# Move operations
RPOPLPUSH source dest             # Pop and push
LMOVE source dest LEFT|RIGHT LEFT|RIGHT  # Move with direction
```
