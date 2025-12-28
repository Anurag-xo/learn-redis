# Redis

## What is Redis?

Redis (REmote DIctionary Server) is an open-source, BSD-licensed **in-memory data structure store** that serves multiple purposes:

- **Database** - Persistent data storage
- **Cache** - High-speed data caching layer
- **Message Broker** - Pub/sub messaging system

### Key Characteristics

- **In-memory storage** for ultra-fast data access
- **Key-value store** architecture
- Built-in **replication** for high availability
- **LUA scripting** support for complex operations
- Multiple levels of **data persistence** options

---

## Redis vs Other NoSQL Databases

### Similarities

Redis shares some characteristics with NoSQL databases like MongoDB and CouchDB in terms of data storage patterns and flexibility.

### Key Differences

| Feature         | Redis                                     | MongoDB/CouchDB                    |
| --------------- | ----------------------------------------- | ---------------------------------- |
| **Storage**     | Memory-based                              | Disk-based                         |
| **Speed**       | Extremely fast (microsecond latency)      | Fast (millisecond latency)         |
| **Primary Use** | Cache, session store, real-time analytics | Primary database, document storage |
| **Data Size**   | Limited by RAM                            | Limited by disk space              |

### Common Use Cases

- **Caching** - Reduce database load and improve response times
- **Session management** - Store user session data
- **Real-time analytics** - Leaderboards, counters, metrics
- **Message queuing** - Task queues and pub/sub patterns

---

## Redis Data Types

Redis supports various data structures beyond simple key-value pairs:

### Core Data Types

1. **Strings** - Binary-safe strings up to 512MB
2. **Hashes** - Maps between string fields and values (like objects)
3. **Lists** - Ordered collections of strings
4. **Sets** - Unordered collections of unique strings
5. **Sorted Sets** - Sets ordered by a score value

### Advanced Data Types

6. **Bitmaps** - Bit-level operations on strings
7. **HyperLogLogs** - Probabilistic data structure for cardinality estimation
8. **Geospatial Indexes** - Store and query geographic coordinates
9. **Streams** - Append-only log data structure

---

## Redis CLI (Command Line Interface)

The Redis CLI is a terminal-based client for interacting with Redis servers, allowing you to send commands and receive responses directly.

### Operating Modes

#### 1. Interactive Mode

The default mode with basic line editing capabilities.

```bash
redis-cli
127.0.0.1:6379> SET key value
OK
```

**Features:**

- Command history navigation
- Tab completion
- Multi-line input support

#### 2. Slave Mode

Simulates a Redis replica (slave) to inspect replication streams.

```bash
redis-cli --slave
```

**Use cases:**

- Debugging replication issues
- Monitoring data flow between master and replicas

#### 3. Replication Stream Mode

Displays the replication stream in a human-readable format.

```bash
redis-cli --rdb /path/to/dump.rdb
```

**Use cases:**

- Analyzing replication data
- Troubleshooting synchronization issues
- Understanding data transfer patterns

### CLI Connection Options

```bash
# Connect to local Redis (default: localhost:6379)
redis-cli

# Connect to remote Redis server
redis-cli -h hostname -p port

# Authenticate with password
redis-cli -a password

# Select specific database
redis-cli -n database_number
```

---

## Quick Start Example

```bash
# Start Redis CLI
redis-cli

# Test connection
PING
# Output: PONG

# Store data
SET user:1:name "John Doe"
# Output: OK

# Retrieve data
GET user:1:name
# Output: "John Doe"

# Work with lists
LPUSH tasks "task1" "task2" "task3"
LRANGE tasks 0 -1
# Output: 1) "task3" 2) "task2" 3) "task1"
```

---

## When to Use Redis

✅ **Good for:**

- Caching frequently accessed data
- Real-time leaderboards and counters
- Session storage
- Pub/sub messaging
- Rate limiting
- Temporary data with TTL

❌ **Not ideal for:**

- Primary storage of critical data (unless properly configured with persistence)
- Large datasets exceeding available RAM
- Complex querying and relationships (better suited for relational or document databases)
