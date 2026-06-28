# Backpressure: The Art of Saying "Slow Down"

**Topic:** System Design | **Date:** 2026-06-29

---

## 🧒 Like you're 5

Imagine a water pipe system. When the tank at the end is full, water keeps flowing in — the pipe bursts! Backpressure is like a smart valve that senses the tank is full and says "Hey, slow down the flow at the source!" Instead of flooding, the system gracefully reduces input until there's room again.

In software: when a service is overwhelmed, it tells its callers "I'm busy, stop sending so much!" — preventing crashes and cascading failures.

↓ Now let's call it what it really is ↓

---

## 🔧 For the engineer

**Backpressure** is a flow-control mechanism where a downstream consumer signals its upstream producer to slow down data transmission. It's fundamental in stream processing, message queues, and reactive systems.

### Key Concepts

- **Problem it solves:** Producer outpaces consumer → memory exhaustion → OOM crash → cascade
- **Without backpressure:** Unbounded queues grow infinitely, latency spikes, system collapses
- **With backpressure:** Flow self-regulates, bounded buffers, graceful degradation
- **Key principle:** Push back the load to the source — don't absorb and die

### Three Strategies

1. **Drop** — Discard new messages when buffer is full. OK for metrics, bad for transactions.
2. **Buffer** — Store in bounded queue. Block producer when full. Risk: memory pressure.
3. **Sample/Throttle** — Reduce rate at source. Best for real-time streams (video, telemetry).
4. **Propagate upstream** — Signal all the way back to the original source. Most robust, most complex.

### Real-World Implementations

- **Reactive Streams (Akka, Project Reactor):** Subscriber requests N items; producer sends exactly N. Pull-based model.
- **TCP Flow Control:** Receiver advertises window size. Sender stops when window = 0. The OG backpressure.
- **Kafka Consumer Lag:** Consumer falls behind → lag grows → you scale consumers or accept delay.
- **gRPC Streaming:** HTTP/2 flow control + application-level credit-based backpressure.

### Code: Simple Backpressure with Semaphore

```javascript
// Producer respects consumer capacity
const MAX_IN_FLIGHT = 10;
const semaphore = new Semaphore(MAX_IN_FLIGHT);

async function produce(items) {
  for (const item of items) {
    await semaphore.acquire();  // blocks when full
    send(item).finally(() => semaphore.release());
  }
}
```

### Backpressure in Inference Pipelines

In GPU inference serving (like Triton/NVCF), backpressure prevents:
- Request queue overflow when GPU is at 100% utilization
- Batch timeout violations from queuing delay
- Cascading timeouts across microservice chains

Pattern: admission control → bounded queue → reject with 429/503 when saturated → client retries with exponential backoff.

---

## ⚡ Micro-action (5 minutes)

Pick one service you own or interact with. Ask: *"What happens when this gets 10x normal traffic?"* Check if it has bounded queues, rate limiting, or backpressure signals. If not, note it as a risk. One sentence is enough — just build the habit of spotting missing backpressure.

---

System Design 💫 Small steps. Every day.
