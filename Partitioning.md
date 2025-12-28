# Redis Partitioning

## What is Partitioning?

Partitioning is the process of **splitting your data across multiple Redis instances**, where each instance stores only a subset of your keys.

---

## Benefits of Partitioning

### 1. **Increased Memory Capacity**

Combines the memory of multiple machines to store larger datasets that exceed the RAM of a single server.

**Example:**

- 3 servers with 16GB RAM each = 48GB total Redis capacity
- Without partitioning: limited to 16GB on one server

### 2. **Horizontal Scalability**

Distributes computational load across multiple CPU cores and servers, improving overall throughput and performance.

**Advantages:**

- Handle more operations per second
- Reduce latency by distributing queries
- Scale infrastructure as data grows

---

## Partitioning Strategies

### 1. **Range Partitioning**

Assigns keys to instances based on ranges (e.g., user IDs 1-1000 → Instance 1, 1001-2000 → Instance 2).

**Pros:** Simple to implement  
**Cons:** Can lead to unbalanced distribution

### 2. **Hash Partitioning**

Uses a hash function on the key to determine which instance stores the data.

**Pros:** Better data distribution  
**Cons:** More complex redistribution when adding/removing nodes

### 3. **Consistent Hashing**

Minimizes data movement when nodes are added or removed.

**Pros:** Efficient scaling  
**Cons:** Requires additional implementation logic

---

## Use Cases

✅ **When to use partitioning:**

- Dataset exceeds single-server memory
- Need to handle millions of requests per second
- Want to distribute load geographically
- Require fault isolation between data segments

⚠️ **Limitations:**

- Operations across multiple keys may not be supported
- More complex backup and monitoring
- Network latency between instances

---

## Implementation Example

```bash
# Instance 1 (handles keys hashing to 0-5461)
redis-server --port 6379

# Instance 2 (handles keys hashing to 5462-10922)
redis-server --port 6380

# Instance 3 (handles keys hashing to 10923-16383)
redis-server --port 6381
```

**Redis Cluster** automates this partitioning with built-in support for data sharding and failover.

---

## Key Takeaways

- Partitioning = **horizontal scaling** for Redis
- Enables databases larger than single-machine RAM
- Distributes both **storage** and **computational load**
- Essential for high-traffic, data-intensive applications
