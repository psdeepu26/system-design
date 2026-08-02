# Request Coalescing: One Fetch, Many Waiters

**Topic:** System Design  
**Category:** Request Coordination  
**Date:** 3 August 2026

---

## 🧒 Like you're 5

Ten children ask a librarian for the same book at the same time. Instead of
sending ten librarians to the shelf, one librarian gets it while everyone else
waits. When the librarian returns, all ten receive the answer.

Services can do this too: identical work arriving together becomes one shared
operation, not a burst of duplicate calls.

---

↓ Turn simultaneous duplicate work into one shared result ↓

## 🔧 For the engineer

**Request coalescing**, also called single-flight, groups concurrent requests by
a stable key. First caller becomes leader and starts work; followers await the
same future or promise. Result—or error—is broadcast, then the in-flight entry
is removed.

| Concept | Design rule |
|---|---|
| **Coalescing key** | Include every field that can change result: resource ID, query shape, tenant, locale, and authorization scope. Two requests may share work only when their outputs are equivalent. |
| **Atomic leader election** | Use an atomic map insertion such as `computeIfAbsent` or a singleflight primitive. A separate “check then create” permits multiple leaders under race. |
| **Lifecycle cleanup** | Delete in-flight state on success, error, and panic. Keep result in a real cache if reuse after completion is wanted; in-flight registry is not the cache. |
| **Failure semantics** | Followers usually share leader's error. Add retries outside the coalescer with jitter and limits, or one failure can immediately trigger another synchronized wave. |
| **Cancellation** | One follower timing out must not cancel work needed by others. Cancel underlying call only when no waiters remain or shared deadline expires. |
| **Control the blast radius** | Cap waiters per key and total in-flight keys. Hot keys need timeout, load shedding, and stale-cache fallback so a shared wait does not become a shared outage. |

**Flow:** Build safe key → Join/create flight → Leader fetches once → Broadcast result → Remove flight

```text
result = singleflight.do(key, async () => {
  return await origin.fetch(resource, sharedDeadline)
})

// Observe: requests, flights, waiters, shared errors
coalescing_ratio = 1 - (flights / requests)
```

> **Boundary:** Coalescing reduces concurrent duplicate work; caching reuses
> completed work across time. Use both for hot reads. Never coalesce requests
> across tenants or permission scopes merely because their URLs match.

---

## ⚡ Micro-action

Find one read path that fans out to a costly dependency. Write its safe
coalescing key, then add two counters: incoming requests and actual dependency
calls. Their difference shows duplicate work you could collapse.

**System Design · Request Coordination**  
💫 Small steps. Every day.
