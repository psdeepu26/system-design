# ⚖️ Load Balancing Algorithms

## 🧒 Like You're 5

Imagine a **busy food court** with 4 counters and 1 long queue. You need a system to decide which counter each person goes to:

- **🔄 Round Robin** — "Next!" Person #1 → Counter A, #2 → B, #3 → C, #4 → D, #5 back to A. Fair rotation, but if Counter A's cook is slow, that queue builds up.
- **🎯 Least Connections** — Look at which counter has the shortest line and send the next person there.
- **⏱️ Least Response Time** — Same as above, plus factor in which counter served fastest.
- **🔒 IP Hash** — Same person, same counter every time so the cook remembers your order.

> **The big idea:** Without a good system, one counter is crushed while three are idle. A load balancer is just a **smart host** making sure everyone gets served fairly.

---

## 🔧 For the Engineer

### Round Robin
Requests in circular order: `A → B → C → A → B → C`. Simple, stateless. **Weighted variant:** servers with more capacity get more requests.

### Least Connections
Route to server with fewest active connections. Adapts to varying request durations.
- Nginx: `upstream backend { least_conn; ... }`
- HAProxy: `balance leastconn`

### Least Response Time
Route to server with lowest avg response time + fewest active connections.
- AWS ALB: "least outstanding requests" (LOR)
- HAProxy: `balance first`

### IP Hash / Sticky Sessions
`hash = crc32(client_ip) % server_count` — same client → same server.
- Nginx: `ip_hash;`
- Best for in-memory sessions without cookies

### Layer 4 vs Layer 7
| Type | Sees | Speed |
|------|------|-------|
| L4 (TCP) | IP + port only | Fast |
| L7 (HTTP) | Headers, cookies, paths | Slower but smarter |

### Real-World Tools
- **NGINX** — Layer 7, great for HTTP APIs
- **HAProxy** — TCP + HTTP, battle-tested
- **AWS ALB** — Managed L7, path/host-based routing
- **K8s Service** — iptables round robin by default
- **Envoy** — Advanced LB + circuit breaking

---

⚡ **Micro-action:** Run `kubectl describe svc <your-service>` to see what LB algorithm your K8s cluster uses.