# Redis Set Commands

## Overview

Sets in Redis are **unordered collections of unique strings**. They are perfect for storing distinct values and performing set operations like unions, intersections, and differences.

---

## Set Characteristics

- **Unique elements** - No duplicate values allowed
- **Unordered** - Elements have no specific order
- **Fast membership testing** - O(1) to check if element exists
- **Set operations** - Support union, intersection, difference
- **Max elements** - Up to 2^32 - 1 elements (over 4 billion)
- **String values** - All members are strings

---

## Basic Set Commands

### SADD - Add Members to Set

Adds one or more members to a set. Creates the set if it doesn't exist.

**Syntax:** `SADD key member [member ...]`

```bash
SADD tags:post:100 "redis" "database" "nosql"
```

**Output:** `(integer) 3` _(number of members added)_

**Adding duplicates (ignored):**

```bash
SADD tags:post:100 "redis" "cache"
```

**Output:** `(integer) 1` _(only "cache" was new)_

**Single member:**

```bash
SADD users:online "user:1001"
```

**Output:** `(integer) 1`

**Use case:** Tags, categories, unique identifiers

### SMEMBERS - Get All Members

Returns all members of the set.

**Syntax:** `SMEMBERS key`

```bash
SMEMBERS tags:post:100
```

**Output:**

```
1) "redis"
2) "database"
3) "nosql"
4) "cache"
```

**⚠️ Warning:** O(N) operation - use cautiously on large sets

**Empty set:**

```bash
SMEMBERS nonexistent
```

**Output:** `(empty array)`

### SISMEMBER - Check Membership

Checks if a member exists in the set.

**Syntax:** `SISMEMBER key member`

```bash
SISMEMBER tags:post:100 "redis"
```

**Output:** `(integer) 1` _(member exists)_

```bash
SISMEMBER tags:post:100 "python"
```

**Output:** `(integer) 0` _(member doesn't exist)_

**Use case:** Permission checking, feature flags, validation

### SMISMEMBER - Check Multiple Members

Checks if multiple members exist in the set.

**Syntax:** `SMISMEMBER key member [member ...]`

```bash
SMISMEMBER tags:post:100 "redis" "python" "database" "java"
```

**Output:**

```
1) (integer) 1  # redis exists
2) (integer) 0  # python doesn't exist
3) (integer) 1  # database exists
4) (integer) 0  # java doesn't exist
```

**Use case:** Batch permission checks, bulk validation

### SCARD - Get Set Cardinality

Returns the number of members in the set.

**Syntax:** `SCARD key`

```bash
SCARD tags:post:100
```

**Output:** `(integer) 4`

**Non-existent set:**

```bash
SCARD nonexistent
```

**Output:** `(integer) 0`

**Use case:** Counting unique visitors, tags, categories

---

## Removing Members

### SREM - Remove Members

Removes one or more members from the set.

**Syntax:** `SREM key member [member ...]`

```bash
SREM tags:post:100 "nosql"
```

**Output:** `(integer) 1` _(number of members removed)_

**Remove multiple:**

```bash
SREM tags:post:100 "cache" "database" "nonexistent"
```

**Output:** `(integer) 2` _(only existing members removed)_

```bash
SMEMBERS tags:post:100
```

**Output:**

```
1) "redis"
```

### SPOP - Remove and Return Random Member

Removes and returns one or more random members from the set.

**Syntax:** `SPOP key [count]`

```bash
SADD lottery "ticket1" "ticket2" "ticket3" "ticket4" "ticket5"
SPOP lottery
```

**Output:** `"ticket3"` _(random member)_

**Pop multiple:**

```bash
SPOP lottery 2
```

**Output:**

```
1) "ticket1"
2) "ticket5"
```

```bash
SMEMBERS lottery
```

**Output:**

```
1) "ticket2"
2) "ticket4"
```

**Use case:** Random selection, lottery, sampling

### SMOVE - Move Member Between Sets

Atomically moves a member from one set to another.

**Syntax:** `SMOVE source destination member`

```bash
SADD active:users "user1" "user2" "user3"
SADD inactive:users "user4"
SMOVE active:users inactive:users "user2"
```

**Output:** `(integer) 1` _(success)_

```bash
SMEMBERS active:users
```

**Output:**

```
1) "user1"
2) "user3"
```

```bash
SMEMBERS inactive:users
```

**Output:**

```
1) "user4"
2) "user2"
```

**If member doesn't exist:**

```bash
SMOVE active:users inactive:users "user99"
```

**Output:** `(integer) 0` _(failure)_

---

## Random Operations

### SRANDMEMBER - Get Random Member(s) Without Removing

Returns one or more random members without removing them.

**Syntax:** `SRANDMEMBER key [count]`

```bash
SADD colors "red" "blue" "green" "yellow" "purple"
SRANDMEMBER colors
```

**Output:** `"green"` _(random member, still in set)_

**Get multiple unique random members:**

```bash
SRANDMEMBER colors 3
```

**Output:**

```
1) "blue"
2) "yellow"
3) "red"
```

**Negative count (allow repetitions):**

```bash
SRANDMEMBER colors -5
```

**Output:**

```
1) "red"
2) "red"
3) "blue"
4) "yellow"
5) "green"
```

_(may contain duplicates)_

**Use case:** Random recommendations, sampling

---

## Set Operations

### SUNION - Union of Sets

Returns the union of multiple sets (all unique members).

**Syntax:** `SUNION key [key ...]`

```bash
SADD skills:alice "python" "redis" "docker"
SADD skills:bob "java" "redis" "kubernetes"
SUNION skills:alice skills:bob
```

**Output:**

```
1) "python"
2) "redis"
3) "docker"
4) "java"
5) "kubernetes"
```

**Single set:**

```bash
SUNION skills:alice
```

**Output:**

```
1) "python"
2) "redis"
3) "docker"
```

### SUNIONSTORE - Store Union Result

Stores the union of multiple sets in a destination key.

**Syntax:** `SUNIONSTORE destination key [key ...]`

```bash
SUNIONSTORE skills:all skills:alice skills:bob
```

**Output:** `(integer) 5` _(number of members in result)_

```bash
SMEMBERS skills:all
```

**Output:**

```
1) "python"
2) "redis"
3) "docker"
4) "java"
5) "kubernetes"
```

**Use case:** Combining categories, merging tags, aggregating data

### SINTER - Intersection of Sets

Returns the intersection of multiple sets (common members only).

**Syntax:** `SINTER key [key ...]`

```bash
SADD project:team1 "alice" "bob" "charlie"
SADD project:team2 "bob" "dave" "alice"
SINTER project:team1 project:team2
```

**Output:**

```
1) "alice"
2) "bob"
```

**Multiple sets:**

```bash
SADD project:team3 "alice" "eve"
SINTER project:team1 project:team2 project:team3
```

**Output:**

```
1) "alice"
```

**No intersection:**

```bash
SADD setA "a" "b"
SADD setB "c" "d"
SINTER setA setB
```

**Output:** `(empty array)`

### SINTERSTORE - Store Intersection Result

Stores the intersection of multiple sets in a destination key.

**Syntax:** `SINTERSTORE destination key [key ...]`

```bash
SINTERSTORE common:members project:team1 project:team2
```

**Output:** `(integer) 2` _(number of common members)_

```bash
SMEMBERS common:members
```

**Output:**

```
1) "alice"
2) "bob"
```

**Use case:** Finding common interests, shared permissions

### SINTERCARD - Intersection Cardinality

Returns the number of elements in the intersection without computing it.

**Syntax:** `SINTERCARD numkeys key [key ...] [LIMIT limit]`

```bash
SINTERCARD 2 project:team1 project:team2
```

**Output:** `(integer) 2` _(count of common members)_

**With limit (stop counting after limit):**

```bash
SINTERCARD 3 project:team1 project:team2 project:team3 LIMIT 1
```

**Output:** `(integer) 1` _(stops at limit)_

**Use case:** Efficient counting without storing results

### SDIFF - Difference of Sets

Returns members in the first set that don't exist in subsequent sets.

**Syntax:** `SDIFF key [key ...]`

```bash
SADD premium:features "feature1" "feature2" "feature3" "feature4"
SADD basic:features "feature1" "feature2"
SDIFF premium:features basic:features
```

**Output:**

```
1) "feature3"
2) "feature4"
```

_(features only in premium)_

**Multiple sets:**

```bash
SADD blocked:features "feature3"
SDIFF premium:features basic:features blocked:features
```

**Output:**

```
1) "feature4"
```

**Order matters:**

```bash
SDIFF basic:features premium:features
```

**Output:** `(empty array)` _(all basic features are in premium)_

### SDIFFSTORE - Store Difference Result

Stores the difference of multiple sets in a destination key.

**Syntax:** `SDIFFSTORE destination key [key ...]`

```bash
SDIFFSTORE exclusive:features premium:features basic:features
```

**Output:** `(integer) 2`

```bash
SMEMBERS exclusive:features
```

**Output:**

```
1) "feature3"
2) "feature4"
```

**Use case:** Finding unique items, exclusions, filtering

---

## Scanning and Iteration

### SSCAN - Incrementally Iterate Set Members

Iterates over set members without blocking the server.

**Syntax:** `SSCAN key cursor [MATCH pattern] [COUNT count]`

```bash
# Add many members
SADD large:set item1 item2 item3 item4 item5 item6 item7 item8 item9 item10

# Start scan
SSCAN large:set 0
```

**Output:**

```
1) "0"  # next cursor (0 means complete)
2) 1) "item5"
   2) "item1"
   3) "item9"
   4) "item3"
   5) "item7"
   6) "item2"
   7) "item10"
   8) "item4"
   9) "item6"
   10) "item8"
```

**With pattern matching:**

```bash
SADD products prod:1 prod:2 prod:3 cat:1 cat:2 tag:1
SSCAN products 0 MATCH prod:*
```

**Output:**

```
1) "0"
2) 1) "prod:1"
   2) "prod:2"
   3) "prod:3"
```

**With count hint:**

```bash
SSCAN large:set 0 COUNT 3
```

**Output:** Returns approximately 3 members per iteration

**Use case:** Safe iteration over large sets, pattern filtering

---

## Practical Examples

### Example 1: User Permissions System

```bash
# Define role permissions
SADD role:admin "read" "write" "delete" "manage_users" "view_analytics"
SADD role:editor "read" "write" "delete"
SADD role:viewer "read"

# Assign roles to users
SADD user:1001:roles "editor"
SADD user:1002:roles "admin"
SADD user:1003:roles "viewer"

# Check if user has permission
SISMEMBER role:editor "write"
# Output: (integer) 1

# Get all permissions for role
SMEMBERS role:editor
# Output:
# 1) "read"
# 2) "write"
# 3) "delete"

# Find common permissions between roles
SINTER role:editor role:viewer
# Output:
# 1) "read"

# Find exclusive admin permissions
SDIFF role:admin role:editor
# Output:
# 1) "manage_users"
# 2) "view_analytics"
```

### Example 2: Tagging System

```bash
# Add tags to posts
SADD post:100:tags "redis" "database" "nosql" "cache"
SADD post:101:tags "redis" "tutorial" "beginners"
SADD post:102:tags "database" "sql" "mysql"

# Find posts with specific tag (using multiple sets with tag as key)
SADD tag:redis:posts "100" "101"
SADD tag:database:posts "100" "102"

# Find posts tagged with both redis AND database
SINTER tag:redis:posts tag:database:posts
# Output:
# 1) "100"

# Find posts tagged with redis OR database
SUNION tag:redis:posts tag:database:posts
# Output:
# 1) "100"
# 2) "101"
# 3) "102"

# Find posts tagged with redis but NOT database
SDIFF tag:redis:posts tag:database:posts
# Output:
# 1) "101"

# Count tags on a post
SCARD post:100:tags
# Output: (integer) 4

# Remove a tag
SREM post:100:tags "cache"
```

### Example 3: Online Users Tracking

```bash
# User comes online
SADD users:online "user:1001"
SADD users:online "user:1002" "user:1003"

# Check if user is online
SISMEMBER users:online "user:1001"
# Output: (integer) 1

# Get count of online users
SCARD users:online
# Output: (integer) 3

# Get all online users
SMEMBERS users:online
# Output:
# 1) "user:1001"
# 2) "user:1002"
# 3) "user:1003"

# User goes offline
SREM users:online "user:1002"

# Move user to AFK status
SMOVE users:online users:afk "user:1003"
```

### Example 4: Unique Visitors Tracking

```bash
# Track daily visitors
SADD visitors:2025-12-28 "ip:192.168.1.1" "ip:192.168.1.2" "ip:192.168.1.1"
# Output: (integer) 2 (duplicate ignored)

# Count unique visitors
SCARD visitors:2025-12-28
# Output: (integer) 2

# Track yesterday's visitors
SADD visitors:2025-12-27 "ip:192.168.1.1" "ip:192.168.1.5"

# Find returning visitors (visited both days)
SINTER visitors:2025-12-27 visitors:2025-12-28
# Output:
# 1) "ip:192.168.1.1"

# Find new visitors (today only)
SDIFF visitors:2025-12-28 visitors:2025-12-27
# Output:
# 1) "ip:192.168.1.2"

# Total unique visitors across both days
SUNIONSTORE visitors:total visitors:2025-12-27 visitors:2025-12-28
SCARD visitors:total
# Output: (integer) 3
```

### Example 5: Lottery/Random Selection

```bash
# Add participants
SADD lottery:round1 "user:1" "user:2" "user:3" "user:4" "user:5" "user:6" "user:7" "user:8" "user:9" "user:10"

# Pick 3 winners (removes them)
SPOP lottery:round1 3
# Output:
# 1) "user:7"
# 2) "user:3"
# 3) "user:9"

# Check remaining participants
SCARD lottery:round1
# Output: (integer) 7

# Preview random participants without removing
SRANDMEMBER lottery:round1 2
# Output:
# 1) "user:5"
# 2) "user:2"

# Participants still in set
SCARD lottery:round1
# Output: (integer) 7
```

### Example 6: Following/Followers System

```bash
# User alice follows bob and charlie
SADD user:alice:following "bob" "charlie"
SADD user:bob:followers "alice"
SADD user:charlie:followers "alice"

# User bob follows alice and dave
SADD user:bob:following "alice" "dave"
SADD user:alice:followers "bob"
SADD user:dave:followers "bob"

# Find mutual follows (alice and bob follow each other)
SINTER user:alice:following user:bob:following
# Output:
# (empty array if looking for who they both follow)

# Check if alice follows bob
SISMEMBER user:alice:following "bob"
# Output: (integer) 1

# Find who bob follows but alice doesn't
SDIFF user:bob:following user:alice:following
# Output:
# 1) "dave"

# Get follower count
SCARD user:alice:followers
# Output: (integer) 1

# Get following count
SCARD user:alice:following
# Output: (integer) 2

# Unfollow
SREM user:alice:following "charlie"
SREM user:charlie:followers "alice"
```

### Example 7: Feature Flags per Environment

```bash
# Production features
SADD env:production:enabled "feature_a" "feature_b" "feature_c"

# Staging features (includes experimental)
SADD env:staging:enabled "feature_a" "feature_b" "feature_c" "feature_x" "feature_y"

# Check if feature enabled in production
SISMEMBER env:production:enabled "feature_x"
# Output: (integer) 0

# Find experimental features (staging only)
SDIFF env:staging:enabled env:production:enabled
# Output:
# 1) "feature_x"
# 2) "feature_y"

# Promote feature to production
SMOVE env:staging:enabled env:production:enabled "feature_x"
```

### Example 8: Shopping Categories

```bash
# Products in categories
SADD category:electronics "product:101" "product:102" "product:103"
SADD category:computers "product:101" "product:105"
SADD category:sale "product:102" "product:105" "product:110"

# Find electronics on sale
SINTER category:electronics category:sale
# Output:
# 1) "product:102"

# Find all products in electronics or computers
SUNION category:electronics category:computers
# Output:
# 1) "product:101"
# 2) "product:102"
# 3) "product:103"
# 4) "product:105"

# Count products in category
SCARD category:electronics
# Output: (integer) 3

# Add product to category
SADD category:electronics "product:120"

# Remove product from category
SREM category:sale "product:110"
```

### Example 9: IP Blacklist/Whitelist

```bash
# Blacklisted IPs
SADD ip:blacklist "192.168.1.100" "192.168.1.101" "10.0.0.50"

# Whitelisted IPs
SADD ip:whitelist "192.168.1.200" "192.168.1.201"

# Check if IP is blacklisted
SISMEMBER ip:blacklist "192.168.1.100"
# Output: (integer) 1

# Check if IP is whitelisted
SISMEMBER ip:whitelist "192.168.1.100"
# Output: (integer) 0

# Add IP to blacklist
SADD ip:blacklist "10.0.0.75"

# Remove IP from blacklist
SREM ip:blacklist "192.168.1.100"

# Count blacklisted IPs
SCARD ip:blacklist
# Output: (integer) 3
```

### Example 10: Skills Matching for Job Recommendations

```bash
# Job requirements
SADD job:1001:required "python" "redis" "docker" "kubernetes"
SADD job:1002:required "java" "spring" "mysql"

# User skills
SADD user:alice:skills "python" "redis" "docker" "postgresql"
SADD user:bob:skills "java" "spring" "mysql" "mongodb"

# Find how many required skills alice has for job:1001
SINTERCARD 2 job:1001:required user:alice:skills
# Output: (integer) 3

# Find missing skills
SDIFF job:1001:required user:alice:skills
# Output:
# 1) "kubernetes"

# Find matching skills
SINTER job:1001:required user:alice:skills
# Output:
# 1) "python"
# 2) "redis"
# 3) "docker"

# Check if user has all required skills
SDIFF job:1002:required user:bob:skills
# Output: (empty array) - bob has all skills!
```

---

## Command Comparison Table

| Command       | Purpose            | Modifies Set | Returns                  |
| ------------- | ------------------ | ------------ | ------------------------ |
| `SADD`        | Add member(s)      | Yes          | Count of added members   |
| `SREM`        | Remove member(s)   | Yes          | Count of removed members |
| `SMEMBERS`    | Get all members    | No           | Array of members         |
| `SISMEMBER`   | Check membership   | No           | 1 (exists) or 0 (not)    |
| `SMISMEMBER`  | Check multiple     | No           | Array of 1s and 0s       |
| `SCARD`       | Get member count   | No           | Integer count            |
| `SPOP`        | Remove random      | Yes          | Member(s) removed        |
| `SRANDMEMBER` | Get random         | No           | Random member(s)         |
| `SMOVE`       | Move between sets  | Yes          | 1 (success) or 0 (fail)  |
| `SUNION`      | Union of sets      | No           | Array of all members     |
| `SUNIONSTORE` | Store union        | Yes (dest)   | Count in destination     |
| `SINTER`      | Intersection       | No           | Array of common members  |
| `SINTERSTORE` | Store intersection | Yes (dest)   | Count in destination     |
| `SINTERCARD`  | Count intersection | No           | Integer count            |
| `SDIFF`       | Difference         | No           | Array of unique members  |
| `SDIFFSTORE`  | Store difference   | Yes (dest)   | Count in destination     |
| `SSCAN`       | Iterate members    | No           | Cursor + members         |

---

## Performance Characteristics

| Operation     | Time Complexity | Notes                            |
| ------------- | --------------- | -------------------------------- |
| `SADD`        | O(1)            | Per member                       |
| `SREM`        | O(N)            | N = members to remove            |
| `SISMEMBER`   | O(1)            | Hash table lookup                |
| `SMISMEMBER`  | O(N)            | N = members to check             |
| `SCARD`       | O(1)            | Cached value                     |
| `SMEMBERS`    | O(N)            | N = set size                     |
| `SPOP`        | O(1)            | Per member                       |
| `SRANDMEMBER` | O(1)            | Per member                       |
| `SMOVE`       | O(1)            | Atomic operation                 |
| `SUNION`      | O(N)            | N = total members in all sets    |
| `SINTER`      | O(N\*M)         | N = smallest set, M = sets count |
| `SDIFF`       | O(N)            | N = total members in all sets    |
| `SSCAN`       | O(1)            | Per iteration call               |

---

## Best Practices

✅ **Do:**

- Use sets for unique collections (tags, permissions, IDs)
- Use `SISMEMBER` for fast membership checks
- Use set operations for filtering and recommendations
- Use `SSCAN` for iterating large sets safely
- Store set intersections/unions if queried frequently
- Use `SINTERCARD` when you only need the count

❌ **Don't:**

- Use sets for ordered data (use sorted sets instead)
- Use `SMEMBERS` frequently on very large sets
- Store millions of tiny sets (memory inefficient)
- Forget that sets are unordered
- Use sets when duplicates are needed (use lists)
- Perform complex set operations on huge sets in real-time

---

## Common Patterns

### Membership Testing

```bash
SISMEMBER set member  # O(1) fast check
```

### Unique Collection

```bash
SADD unique:items item1 item2 item1  # Duplicates ignored
```

### Random Selection

```bash
SRANDMEMBER pool count  # Non-destructive
SPOP pool count  # Destructive
```

### Set Algebra

```bash
SINTER set1 set2  # Common elements
SUNION set1 set2  # All elements
SDIFF set1 set2   # Elements only in set1
```

---

## Quick Reference

```bash
# Basic operations
SADD key member [member ...]          # Add members
SREM key member [member ...]          # Remove members
SMEMBERS key                          # Get all members
SCARD key                             # Count members

# Membership checks
SISMEMBER key member                  # Check single member
SMISMEMBER key member [member ...]    # Check multiple members

# Random operations
SPOP key [count]                      # Remove and return random
SRANDMEMBER key [count]               # Get random (keep in set)
SMOVE source dest member              # Move member

# Set operations
SUNION key [key ...]                  # Union
SUNIONSTORE dest key [key ...]        # Store union
SINTER key [key ...]                  # Intersection
SINTERSTORE dest key [key ...]        # Store intersection
SINTERCARD numkeys key [key ...]      # Count intersection
SDIFF key [key ...]                   # Difference
SDIFFSTORE dest key [key ...]         # Store difference

# Iteration
SSCAN key cursor [MATCH pattern] [COUNT count]  # Iterate safely
```

---

# Redis Sorted Set Commands

## Overview

Sorted Sets in Redis are **collections of unique strings (members) ordered by an associated score**. Every member has a score, and members are always sorted by this score. They combine the properties of both sets and ordered lists.

---

## Sorted Set Characteristics

- **Unique members** - No duplicate members (like sets)
- **Ordered by score** - Members sorted by score value
- **Score-based ranking** - Access by rank or score range
- **Fast operations** - O(log N) add/remove operations
- **Max elements** - Up to 2^32 - 1 members
- **Double precision scores** - IEEE 754 floating point numbers

---

## Basic Sorted Set Commands

### ZADD - Add Members with Scores

Adds one or more members with scores to a sorted set.

**Syntax:** `ZADD key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]`

```bash
ZADD leaderboard 1500 "alice" 1800 "bob" 1200 "charlie"
```

**Output:** `(integer) 3` _(members added)_

**Single member:**

```bash
ZADD leaderboard 2000 "dave"
```

**Output:** `(integer) 1`

**Update existing member's score:**

```bash
ZADD leaderboard 1900 "alice"
```

**Output:** `(integer) 0` _(member existed, score updated)_

**Options:**

- `NX` - Only add new members (don't update existing)
- `XX` - Only update existing members (don't add new)
- `GT` - Only update if new score is greater
- `LT` - Only update if new score is less
- `CH` - Return count of changed elements (not just added)
- `INCR` - Increment score instead of setting it

**Examples with options:**

```bash
ZADD leaderboard NX 1600 "alice"
# Output: (integer) 0 (alice exists, not added)

ZADD leaderboard XX 1700 "eve"
# Output: (integer) 0 (eve doesn't exist, not added)

ZADD leaderboard GT 2100 "bob"
# Output: (integer) 0 (updated if 2100 > current score)

ZADD leaderboard INCR 50 "charlie"
# Output: "1250" (incremented score)
```

### ZSCORE - Get Member's Score

Returns the score of a member in the sorted set.

**Syntax:** `ZSCORE key member`

```bash
ZSCORE leaderboard "bob"
```

**Output:** `"1800"`

```bash
ZSCORE leaderboard "alice"
```

**Output:** `"1900"`

**Non-existent member:**

```bash
ZSCORE leaderboard "unknown"
```

**Output:** `(nil)`

### ZCARD - Get Sorted Set Cardinality

Returns the number of members in the sorted set.

**Syntax:** `ZCARD key`

```bash
ZCARD leaderboard
```

**Output:** `(integer) 4`

**Non-existent key:**

```bash
ZCARD nonexistent
```

**Output:** `(integer) 0`

### ZCOUNT - Count Members in Score Range

Returns the count of members with scores within the given range.

**Syntax:** `ZCOUNT key min max`

```bash
ZCOUNT leaderboard 1200 1800
```

**Output:** `(integer) 3` _(charlie:1250, alice:1900, bob:1800)_

**Exclusive ranges (use parentheses):**

```bash
ZCOUNT leaderboard (1200 1800
# Output: (integer) 2 (excludes 1200)

ZCOUNT leaderboard 1200 (1800
# Output: (integer) 2 (excludes 1800)

ZCOUNT leaderboard (1200 (1800
# Output: (integer) 1 (excludes both)
```

**Infinity:**

```bash
ZCOUNT leaderboard -inf +inf
# Output: (integer) 4 (all members)

ZCOUNT leaderboard 1500 +inf
# Output: (integer) 3 (scores >= 1500)
```

---

## Retrieving Members by Rank

### ZRANGE - Get Members by Rank (Ascending)

Returns members in the specified rank range, ordered from lowest to highest score.

**Syntax:** `ZRANGE key start stop [WITHSCORES] [REV] [BYSCORE|BYLEX] [LIMIT offset count]`

```bash
ZRANGE leaderboard 0 -1
```

**Output:**

```
1) "charlie"   # lowest score (1250)
2) "alice"     # 1900
3) "bob"       # 1800
4) "dave"      # highest score (2000)
```

**With scores:**

```bash
ZRANGE leaderboard 0 -1 WITHSCORES
```

**Output:**

```
1) "charlie"
2) "1250"
3) "alice"
4) "1900"
5) "bob"
6) "1800"
7) "dave"
8) "2000"
```

**Get top 2:**

```bash
ZRANGE leaderboard 0 1
```

**Output:**

```
1
```
