# ADR-001: Funnel-Engine-Lens as the operating model

**Status:** Accepted · **Date:** 2026-03-12

## Context

Production LLM serving in 2026 has three operational concerns that don't compose well in existing platforms:

1. **Funnel** — request shaping: routing, batching, rate-limiting, retries, fallbacks across runtimes and providers
2. **Engine** — model execution: which runtime (vLLM / TGI / Triton / SGLang / llama.cpp), which GPU, which scheduler
3. **Lens** — observability: latency percentiles, token cost, fairness, drift, lineage

Existing platforms collapse two of these into one. Seldon Core does Engine + Lens but pushes Funnel onto the application. KServe does Engine well, weak on Funnel and Lens. LiteLLM does Funnel, no Engine, no Lens. Kong/Envoy do Funnel, blind to LLM internals.

The cost of collapsing concerns is operational: you can't tune Funnel decisions without Lens signal, and you can't make Engine decisions without Funnel context.

## Decision

Pyxis treats the three as **independent layers with named interfaces** between them:

- **Funnel → Engine** contract: structured request envelope including SLA hints, fallback policy, tenancy
- **Engine → Lens** contract: per-request signal stream including TTFT, TPOT, batch position, KV-cache pressure
- **Lens → Funnel** contract: feedback loop including per-runtime health, per-tenant pressure, drift signal

Each layer can be swapped independently. The Engine in particular is pluggable — that's the model-agnostic part.

## Consequences

- More moving parts than collapsed platforms — three deployments rather than one
- Wins on operability: each layer can be debugged, scaled, replaced in isolation
- Wins on portability: changing inference runtime touches only the Engine adapter
- Cost: need to define the interfaces tightly and resist scope-creep across them

## Related

- See [`docs/adr-002-runtime-adapter-abstraction.md`](./adr-002-runtime-adapter-abstraction.md) for how the Engine layer pluggability works in detail
