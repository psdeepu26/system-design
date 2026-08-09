# Hedged Requests: Race the Tail Without Melting the Backend

**Topic:** 🏗️ System Design  
**Category:** Latency Resilience  
**Date:** 10 August 2026

## 🧒 Like you're 5

You ask one friend to bring a glass of water. Usually they return quickly, but today they get distracted. After waiting a little—not immediately—you ask a second friend. You use the first glass that arrives and tell the other friend to stop.

That second request is a **hedge**. It prevents one unusually slow helper from making you wait forever. But asking two friends every time would waste effort and crowd the kitchen.

## ↓ Now call it what it really is ↓

## 🔧 For the engineer

A hedged request sends a duplicate only after the original exceeds a delay threshold. The client accepts the first successful response and cancels or ignores the loser. This trades controlled extra load for lower tail latency.

### Trigger at a percentile

Start the hedge near recent p95 or p99 latency, not at time zero. Most fast requests remain single-shot; only stragglers create duplicates.

### Choose another replica

Route the hedge to a different healthy replica, zone, connection, or cache path. Repeating the same bottleneck provides little benefit.

### First success wins

Return the first valid result. Cancel the loser when the protocol supports cancellation; otherwise discard its response and account for its work.

### Idempotency boundary

Hedge reads and naturally idempotent operations. For writes, use an idempotency key or avoid hedging—two executions can create two side effects.

### Load budget

Cap hedge rate globally and per caller. A 2% hedge rate may be safe; duplicating 100% of requests can create the latency problem it tries to solve.

### Adaptive guardrails

Disable hedging when error rate, queue depth, or backend utilization crosses a limit. During overload, load shedding beats duplication.

```text
t = 0          Send primary
t = p95        Send hedge if primary remains pending
first success  Return response and cancel loser
```

```python
if primary.pending_for > hedge_delay \
   and hedge_budget.try_acquire() \
   and backend.utilization < 0.70:
    hedge = send_to(other_replica)
    return first_success(primary, hedge)
```

**Key equation:** extra load ≈ request rate × fraction still pending at hedge delay. Measure that fraction before enabling hedging.

Track p50/p95/p99 latency, hedge rate, win rate, cancellation success, extra backend work, and error rate. A low hedge win rate means the delay is too short, replicas share the same bottleneck, or the tail is not replica-specific.

## ⚡ Micro-action (5 minutes)

1. Pick one read-only RPC with painful p99 latency.
2. Find its current p95 latency and fraction of requests still running at that point.
3. Estimate extra load if that fraction receives one hedge.
4. Write one safety rule: “Hedge only when utilization is below ___% and cap duplicates at ___%.”

---

🏗️ System Design · 💫 Small steps. Every day.
