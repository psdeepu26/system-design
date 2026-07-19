# Tail Latency: Why the Slowest 1% Shapes the Whole System

**Topic:** System Design · Performance  
**Date:** 20 July 2026

## 🧒 Like you're 5

Imagine 100 children lining up for lunch. Ninety-nine get food quickly, but one waits ten minutes. The average wait still looks fine. Yet that child experiences a broken cafeteria—and a class needing several lunch counters must wait for the slowest counter before everyone can eat.

## ↓ Now let's call it what it really is ↓

## 🔧 For the engineer

**Tail latency** is response time at high percentiles such as p95, p99, or p99.9. An average hides outliers. In fan-out systems, those outliers become normal request-path behavior.

- **Percentiles:** p99 means 99% of requests finish at or below this time; 1% take longer. It says nothing about how slow that final 1% becomes.
- **Fan-out amplification:** If one user request calls 100 independent shards, each with a 1% slow-request chance, probability of at least one slow shard is `1 − 0.99^100 ≈ 63.4%`.
- **Common causes:** Queueing, GC pauses, lock contention, cold caches, noisy neighbors, retries, overloaded dependencies, and network retransmits.
- **Measurement rule:** Track percentiles per endpoint, region, dependency, and workload class. Never average percentiles across instances; aggregate histograms instead.
- **Mitigations:** Bound queues, set deadline budgets, cancel obsolete work, warm caches, isolate pools, shed load, reduce fan-out, and use hedged requests selectively.
- **Trade-off:** Hedging can lower tails but increase load. Trigger only after a delay, cap duplicates, and use it for idempotent operations.

### Deadline budget example

```text
User SLO budget:       500 ms
Gateway + network:      50 ms
Service processing:    150 ms
Dependency budget:     250 ms
Safety margin:          50 ms

Rule: propagate remaining deadline; do not give every hop 500 ms.
```

**Design smell:** “Average latency is healthy” while timeout rate rises. Inspect p99, queue depth, and fan-out width together.

## ⚡ Micro-action (5 minutes)

Pick one important endpoint. Write down its p50, p95, p99, timeout, and fan-out count. If any value is unknown, add one dashboard query or TODO. Then estimate tail amplification with `1 − (1 − p)^n`.

---

System Design · Performance  
💫 Small steps. Every day.
