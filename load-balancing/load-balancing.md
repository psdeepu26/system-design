# ⚖️ Load Balancing Algorithms

**System Design** · How requests get distributed across servers

---

### 🔄 Round Robin
Requests go to each server in turn: A → B → C → A → B → C...
- ✅ Simple, predictable
- ❌ No awareness of server load or health

### 🎯 Least Connections
Send request to the server with the fewest active connections.
- ✅ Adapts to varying workloads
- ✅ Good for long-lived connections

### ⏱️ Least Response Time
Route to the server with the fastest recent response time.
- ✅ Picks the fastest + least loaded
- ❌ Requires continuous monitoring

### 🧮 IP Hash
Hash the client IP → same client always hits the same server.
- ✅ Built-in session affinity
- ❌ Uneven if IPs cluster behind NAT

---

| Algorithm | Best For | Downside |
|-----------|----------|----------|
| Round Robin | Identical servers | No load awareness |
| Least Connections | Variable request duration | More bookkeeping |
| Least Response Time | Mixed performance | Monitoring overhead |
| IP Hash | Session affinity | Uneven distribution |

---

⚡ **Micro-action:** Check what LB algorithm your current infra uses.