# ADR-002: Runtime adapter abstraction

**Status:** Accepted · **Date:** 2026-03-18

## Context

The Engine layer needs to support multiple inference runtimes (vLLM, TGI, Triton, SGLang, llama.cpp, Ollama). Each runtime has different:

- Wire format (vLLM/TGI use OpenAI-compatible; Triton uses gRPC/HTTP with Triton-native payload; llama.cpp/Ollama have their own)
- Streaming protocol (SSE in OpenAI-compatible; gRPC streams in Triton; chunked HTTP in others)
- Health-check semantics (`/v1/models`, `/health`, `/v2/health/ready`, custom)
- Scheduler primitives (continuous batching vs static, paged-attention vs not, speculative decoding support)
- KV-cache eviction policy

Treating these uniformly is impossible. Treating them as completely separate makes the Funnel layer combinatorially complex.

## Decision

Each runtime gets a **typed adapter** that conforms to the Pyxis `Runtime` interface:

```go
type Runtime interface {
    Probe(ctx context.Context) (RuntimeInfo, error)  // models, version, capabilities
    Generate(ctx context.Context, req Request) (<-chan Token, error)
    Health(ctx context.Context) (HealthState, error)
    SchedulerHints() SchedulerHints  // batch size, prefill/decode, KV memory
}
```

The Funnel layer talks to `Runtime` only — it never knows which concrete runtime it's targeting. New runtimes are added by implementing the interface (≈300-500 LOC per runtime adapter).

## Consequences

- Adding a new runtime is bounded work — known interface surface, known test set
- Adapter authors can lean on the canonical OpenAI-compatible shape via the `vllm-bench` test suite
- Cost: the lowest-common-denominator interface loses runtime-specific features (e.g. Triton's GPU-side state sharing) unless explicitly added to `SchedulerHints`

## Related

- See [`docs/adr-003-fair-share-gpu-scheduling.md`](./adr-003-fair-share-gpu-scheduling.md) for how `SchedulerHints` feeds the scheduling layer
