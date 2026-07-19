# Load Shedding: Protect the Core by Refusing Work

**Topic:** System Design  
**Category:** Resilience  
**Date:** 20 July 2026

## 🧒 Like you're 5

Imagine a small clinic with one doctor. If everyone rushes in, the doctor becomes slower and nobody gets help. A smart receptionist temporarily says “please come back later” to some people so the doctor can keep helping the patients already inside.

↓ Now let's call it what it really is ↓

## 🔧 For the engineer

**Load shedding** deliberately rejects, delays, or degrades work when capacity is exhausted. It is a resilience mechanism: controlled failure is safer than letting queues grow until every request times out.

- **Admission control:** Accept work only when concurrency, queue depth, CPU, or dependency budget is below a safe threshold.
- **Prioritization:** Preserve health checks, writes, and high-value requests; shed bulk, retries, previews, or stale reads first.
- **Fail fast:** Return a clear `429` or `503` with `Retry-After` instead of holding a connection until timeout.
- **Graceful degradation:** Serve cached data, fewer results, or an approximate answer when the full path is overloaded.
- **Retry interaction:** Rejections must be visible to clients. Use bounded exponential backoff and jitter; never create a retry storm.
- **Signal quality:** Alert on shed rate, queue age, saturation, and user-visible success—not only process health.

### Simple admission-control sketch

```text
if in_flight >= MAX_CONCURRENCY:
    return 503, {"Retry-After": "2"}

if request.priority == "bulk" and queue_age > 1s:
    return cached_or_degraded_response()

return process(request)
```

**Design smell:** every client retries immediately after a timeout. Put the refusal point before the expensive work, and make the overload policy explicit in the contract.

## ⚡ Micro-action

Choose one critical service and write its overload policy in three lines: what is protected, what is shed first, and what status/retry signal clients receive. If you cannot answer, the system currently has an implicit—and probably inconsistent—policy.

**System Design · Resilience**  
💫 Small steps. Every day.
