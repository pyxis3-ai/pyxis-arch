# ADR-003: Fair-share GPU scheduling for multi-tenant LLM serving

**Status:** Accepted · **Date:** 2026-04-02

## Context

Multi-tenant LLM serving on shared GPUs has a classic resource-allocation problem with LLM-specific twists:

- A single long-prompt prefill can monopolise the GPU for seconds
- A bursty tenant can starve neighbours by saturating the continuous-batching slot pool
- Naive round-robin produces fairness in *request count* but unfairness in *token count*
- Naive token-share produces fairness in tokens but unfairness in latency (long prompts get throttled exactly when latency-sensitive)

Kubernetes' built-in `ResourceQuota` operates on CPU/memory/storage — it has no notion of GPU-second or token budget.

## Decision

Pyxis defines **fair-share at the token-second granularity** with per-tenant priority bands:

- Each tenant has a `TokenBudget` (tokens/min) and a `LatencyBand` (`real-time` / `bulk` / `batch`)
- The scheduler emits decisions on `(request, runtime-slot, scheduled-time)` tuples
- KEDA scales replicas based on per-tenant queue depth, weighted by `LatencyBand`
- When the cluster is saturated, `bulk` and `batch` workloads are throttled before `real-time`

The scheduler exposes per-tenant pressure to the Lens layer for observability.

## Consequences

- Scheduling decisions are *visible*: each tenant sees their own pressure curve in the Lens dashboard
- Replicating the scheduler requires KEDA + a custom external metric source — non-trivial but not exotic
- Cost: fair-share at the token-second is approximate (depends on prefill estimates); we expose the approximation explicitly in the dashboard

## Related

- See [`docs/adr-004-scale-to-zero-economics.md`](./adr-004-scale-to-zero-economics.md) for how the scheduler interacts with cold-start economics
