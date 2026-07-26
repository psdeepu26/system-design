# Idempotency Keys: Make Retries Safe

**Topic:** System Design  
**Category:** API Reliability  
**Date:** 27 July 2026

---

## 🧒 Like you're 5

You press an elevator button, but its light does not turn on. You press it three
more times. A good elevator still sends only one car—not four.

Network requests need the same protection. When a reply gets lost, a client
must retry without accidentally charging a card twice, creating two orders, or
launching duplicate work.

---

↓ Turn repeated intent into one durable result ↓

## 🔧 For the engineer

An **idempotency key** identifies one logical operation across retries. Server
stores first operation's state and response; later requests with same key return
that result instead of repeating side effects.

| Concept | Design rule |
|---|---|
| **Scope** | Bind key to tenant, endpoint, and operation. `(tenant_id, route, key)` avoids collisions across customers or APIs. |
| **Request fingerprint** | Hash canonical request fields. Same key plus different payload must return `409 Conflict`, not silently reuse old result. |
| **Atomic reservation** | Create key record with a unique constraint before side effect. A check-then-insert race can still execute duplicate work. |
| **Concurrent duplicates** | Represent `IN_PROGRESS`, `SUCCEEDED`, and retryable failure states. Duplicate callers wait briefly or receive a retry signal. |
| **Response replay** | Persist status code and stable response body or resource ID. Retried caller should observe same logical outcome. |
| **Retention** | TTL must exceed maximum client retry window. Expiring too early makes an old retry look like new work; retaining forever wastes storage. |

**Flow:** Reserve key → Validate fingerprint → Execute once → Store result → Replay

```sql
INSERT INTO idempotency (tenant, key, request_hash, state)
VALUES (:tenant, :key, :hash, 'IN_PROGRESS')
ON CONFLICT (tenant, key) DO NOTHING;

-- Only reservation owner performs side effect.
-- Retries read existing state/result.
```

> **Boundary:** Idempotency keys provide effectively-once API semantics only
> inside designed scope. They do not create magical exactly-once delivery;
> downstream side effects also need transactional coupling, an outbox, or their
> own deduplication key.

---

## ⚡ Micro-action

Pick one POST endpoint that clients may retry. Write four decisions: key scope,
payload-mismatch behavior, in-progress response, and retention TTL. If any
answer is unclear, retries are currently part of failure risk—not recovery
design.

**System Design · API Reliability**  
💫 Small steps. Every day.
