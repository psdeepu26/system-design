# Consistent Hashing: The Magic Ring That Never Breaks

**System Design · Monday, June 8, 2026**

## 🧒 Like You're 5

Imagine you have 10 toy boxes and you want to put your toys in them. You decide: "I'll put each toy in the box that matches its color number." Simple! But what if one box breaks? Now ALL the toys need to find new boxes — chaos!

Now imagine a magic circle of boxes. When one box breaks, only the toys that were in THAT box need to move — and they just go to the next box in the circle. Everyone else stays put. That's consistent hashing!

## 🔧 For the Engineer

Consistent hashing solves a fundamental distributed systems problem: **how do you distribute data across nodes so that adding or removing nodes causes minimal disruption?**

### The Problem with Modulo Hashing

Traditional `hash(key) % N` fails when N changes. If you go from 4 to 5 nodes, ~80% of keys remap. In a caching layer, this means a massive cache stampede.

```python
# Naive approach — breaks on resize
node = hash("user:12345") % num_nodes
# 4 nodes → node 1
# 5 nodes → node 3  (different! everything moves)
```

### The Hash Ring

Both nodes AND keys are hashed onto a virtual ring (0 to 2^32). Each key maps to the next clockwise node. When a node leaves, only its keys shift to the next node. When a node joins, it steals keys from its clockwise neighbor.

```python
# Ring positions (simplified)
Node A → position 100
Node B → position 300
Node C → position 600

# Key "user:123" hashes to 250 → goes to Node B
# Key "user:456" hashes to 500 → goes to Node C
# Node B dies → "user:123" goes to Node C (only 1 key moves!)
```

### Virtual Nodes (VNodes)

Real nodes get multiple positions on the ring (e.g., 150 vnodes each). This ensures uniform distribution even with heterogeneous hardware. More vnodes = better balance, at the cost of slightly more memory for the ring metadata.

```python
# Each physical node gets multiple ring entries
Node A → A#0@100, A#1@450, A#2@780, ... (150 entries)
Node B → B#0@200, B#1@550, B#2@890, ... (150 entries)
```

### Where It's Used

- **Amazon DynamoDB** — partition key routing
- **Cassandra** — token ring for data distribution
- **Memcached/Redis Cluster** — key-to-node mapping
- **CDNs** — request routing to edge nodes
- **Envoy/Nginx** — consistent hash load balancing

### Weighted Consistent Hashing

When nodes have different capacities, assign vnodes proportionally. A 2x bigger node gets 2x more vnodes.

```python
small_node = 100 vnodes  (1x capacity)
large_node = 200 vnodes  (2x capacity)
```

## ⚡ Micro-Action

Implement a hash ring in 5 minutes: create a `HashRing` class with `add_node()`, `remove_node()`, and `get_node(key)` methods. Use `hashlib.md5` for hashing. Test it: add 3 nodes, map 1000 keys, then remove a node and count how many keys remapped. You'll see it's only ~33% — not 100% like modulo hashing.

---
Tags: Distributed Systems, Hashing, Scalability, Caching
