# ADR-004: Scale-to-zero inference economics

**Status:** Accepted · **Date:** 2026-04-15

## Context

Enterprise LLM deployments rarely justify 24/7 GPU-baseline cost. A typical pattern:

- Working-hours peak: 200 RPS, 8 hours/day
- Evening: 20 RPS, 4 hours
- Overnight: 0–2 RPS, 12 hours

A GPU-baseline architecture (one always-on replica per model) wastes ≈60% of the GPU-hours. But cold-start for a multi-GB inference model can be 30-90 seconds — incompatible with a real-time SLA.

## Decision

Pyxis uses **runtime-aware scale-to-zero** with three configurations:

1. **Hot-zero**: KEDA scales pod replicas to 0 when idle. Cold-start = full pod start + model load. Acceptable for `bulk` and `batch` workloads.
2. **Warm-zero**: pod stays running but model weights unloaded; KV-cache evicted. Cold-start = model load only (~5-15s on H100). Acceptable for `bulk`, sometimes `real-time` with prefetch.
3. **Always-warm**: classic always-on. Required for hard real-time SLAs.

Workload classification (`LatencyBand` from ADR-003) determines the default scale-to-zero policy.

Cost model exposed to tenants:
- `Hot-zero` is ≈40% of always-warm cost for typical workloads
- `Warm-zero` is ≈60-70% of always-warm cost
- `Always-warm` is the baseline

## Consequences

- Realistic cost numbers — `bulk` tenants typically halve their inference spend
- The cost-model dashboard becomes a forcing function for tenants to classify their workloads correctly
- Cost: cold-start variance is a real operational concern; the Lens layer surfaces this aggressively
