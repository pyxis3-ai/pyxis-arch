# Roadmap

What Pyxis is working on next, organised by the Funnel-Engine-Lens layers (see [ADR-001](./docs/adr-001-funnel-engine-lens.md)).

## Q2 2026 — Foundation

**Engine layer**
- [ ] Stabilise the `Runtime` interface (ADR-002) — feedback from `vllm-bench` integration
- [ ] vLLM adapter — production-tested via `lens.pyxis3.ai`
- [ ] TGI adapter — production-tested
- [ ] llama.cpp adapter — for CPU-fallback / edge scenarios
- [ ] AWS Inferentia/Trainium adapter — SageMaker variant first (see `docs/adapters/aws-inferentia-trainium.md`)

**Lens layer**
- [x] LLM-endpoint discovery (shipped 2026-05)
- [x] In-browser kubectl exec (shipped 2026-04)
- [ ] Per-tenant token-budget visualisation
- [ ] Drift detection integration with upstream Alibi Detect

**Funnel layer**
- [ ] Public spec for the Funnel→Engine envelope
- [ ] Reference implementation of the fair-share scheduler (ADR-003)

## Q3 2026 — Multi-tenancy hardening

- [ ] KEDA scaler — production validation across mixed-runtime workloads
- [ ] Per-tenant ResourceQuota integration
- [ ] Scale-to-zero variants (ADR-004): hot-zero, warm-zero, always-warm
- [ ] Customer cost dashboard — per-tenant token-second accounting

## Q4 2026 — UK launch

- [ ] Pyxis 3 Ltd incorporated as UK sister entity to Pyxis3 LLC
- [ ] First three engineering hires in London
- [ ] First paid design-partner engagement in production
- [ ] Public reference architecture for UK-regulated industries (financial services, NHS-adjacent)

## Q1 2027 — Open-source GA

- [ ] Reference implementation of the model-agnostic serving control plane released under permissive licence on `pyxis3-ai`
- [ ] Pyxis Operator + supporting CRDs as open-source primitives
- [ ] Commercial control plane as SaaS layer above the OSS primitives

## Continuous

- Architecture decisions captured as ADRs in `docs/`
- Benchmark results updated in `pyxis3-ai/vllm-bench/results/`
- Recipes added to `pyxis3-ai/llm-serving-cookbook/recipes/`
- Curated list maintained at `pyxis3-ai/awesome-model-agnostic-llm`
