# Circuit Breakers: The Safety Valve for Distributed Systems

**Topic:** System Design | **Date:** 2026-06-22 (Monday)

## Like You're 5

Imagine your home's electrical system. When a wire shorts out, a **circuit breaker**
trips — it cuts the power instantly so the house doesn't burn down. It intentionally makes
*one room* go dark to save the *whole building*.

In software, a **circuit breaker** does the same thing: when a downstream service starts
failing, the breaker "opens" and stops sending traffic to it after a threshold of failures.
Instead of every request waiting and hanging (tying up all your threads), it immediately
returns a fallback response — like saying "service is temporarily down, try again later."

After a cooldown, it tries again. If the service recovered, it closes the circuit and
traffic flows normally. If not, it stays open a bit longer. Its job is to fail fast so your
system can survive slow.

## For the Engineer

The circuit breaker pattern implements a **finite state machine** with three states,
gated by configurable thresholds. It sits between your code and the remote service as a
proxy that monitors failure rates in real-time.

### Three States

- **CLOSED (Normal):** All requests pass through to the downstream service. Failures are
  counted against a rolling window threshold. Transitions: When failure rate > threshold
  -> OPEN.
- **OPEN (Tripped):** All requests immediately return a fallback. No network call is made
  — fail fast, consume zero resources. Transitions: After cooldown timeout -> HALF_OPEN.
- **HALF_OPEN (Testing Recovery):** A small number of probe requests are allowed through.
  If they succeed -> CLOSED. If they fail -> OPEN (reset timer).

### Core Configuration Parameters

```java
// Resilience4j-style configuration
CircuitBreakerConfig.custom()
  .failureRateThreshold(50)        // Open if >50% of calls fail
  .slowCallRateThreshold(80)       // Open if >80% are "slow"
  .slowCallDurationThreshold(Duration.ofSeconds(2))
  .slidingWindowSize(10)           // Look at last 10 calls
  .minimumNumberOfCalls(5)         // Don't trip on the first 4 failures
  .waitDurationInOpenState(Duration.ofSeconds(30))
  .permittedNumberOfCallsInHalfOpenState(3)
  .build()
```

### Key Metrics to Alert On

- `circuitBreakerState` transitions — OPEN events are P2 incidents
- `failureRate` — approaching threshold means trouble incoming
- `notPermittedCalls` — high count = service degradation cascading
- `slowCallRate` — often a leading indicator before hard failures

### Fallback Strategies (in order of preference)

1. **Cached response** — return the last known good response (stale is better than unavailable)
2. **Default value** — return a safe default (e.g., empty recommendations, pick-from-last-seen)
3. **Degraded path** — use a simpler service or algorithm that does less but stays up
4. **Queue for later** — write to a queue/Kafka; replay when downstream recovers

### Common Pitfall: Breakers All The Way Down

If Service A calls Service B which calls Service C, each needs its own breaker. Without B's
breaker, C's failure can exhaust B's threads, which then exhausts A's threads — the failure
cascades upstream. **Every outbound connection gets a breaker.**

## Micro-Action

**5-minute audit:** Pick one service you own. Check: does it have circuit breakers on ALL
outbound dependencies?

```bash
# Find outbound service calls without breakers
grep -r "WebClient\|RestTemplate\|HttpClient" --include="*.java" src/ | \
  grep -v -i "circuitbreaker\|circuit.breaker\|CircuitBreaker"
```

If any appear in the output, prioritize adding a breaker to the one called most frequently.
Even a `failureRateThreshold=50` with `slidingWindowSize=10` is better than nothing.
