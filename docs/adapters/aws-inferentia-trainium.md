# AWS Inferentia / Trainium adapter design notes

## Why

Pyxis's runtime-adapter abstraction (ADR-002) treats every inference runtime as plug-replaceable. AWS Inferentia (`Inf1`, `Inf2`) and Trainium (`Trn1`) silicon expose runtimes that don't follow the OpenAI-compatible convention — they ship via SageMaker endpoints or the lower-level Neuron SDK.

The model-agnostic argument breaks if we can't include AWS's specialised silicon. So the Inferentia/Trainium adapter is on the priority list.

## What changes for Inferentia/Trainium

| Concern | OpenAI-compatible (vLLM, TGI, Triton) | Inferentia/Trainium |
|---|---|---|
| Wire format | `POST /v1/chat/completions` with JSON | Neuron Runtime gRPC or SageMaker `InvokeEndpoint` |
| Streaming | Server-Sent Events (SSE) | SageMaker has streaming variant; Neuron Runtime is request-response |
| Batching | Continuous (vLLM) or static (Triton) | Static batching from Neuron compiler; dynamic batching via SageMaker async-inference |
| Model loading | `.safetensors` / `.gguf` weights | Neuron-compiled artefact (.neff) — compile-time fixed shapes |
| Quantisation | Runtime-side | Compile-time |
| KV-cache | Runtime-managed (vLLM PagedAttention etc.) | Sequence-dimension-aware on Neuron — managed at compile time |

## Adapter strategy

Two paths in parallel:

**1. SageMaker endpoint wrapper** — wraps `InvokeEndpoint` / `InvokeEndpointWithResponseStream` behind the standard Pyxis `Runtime` interface. Lower-friction integration; loses control over batching. Use case: customer already has Inferentia/Trainium deployed via SageMaker and wants to put Pyxis in front.

**2. Neuron Runtime gRPC adapter** — talks directly to Neuron Runtime via the gRPC API. More control; need to handle Neuron-compile-time constraints (fixed batch shapes, no on-the-fly quantisation changes). Use case: customer wants direct hardware control + scale-to-zero.

Both adapters expose the same `Runtime` interface (per ADR-002). The Funnel layer doesn't see the difference.

## Constraints surfaced by the adapter design

- Inferentia/Trainium compile-time fixed shapes mean the `SchedulerHints` API needs a per-compile-config dimension. This is being added to the `Runtime.SchedulerHints()` return type.
- SageMaker async-inference has minimum batch-size assumptions that don't match Pyxis's request-by-request model. The SageMaker adapter buffers requests up to the configured batch shape.
- KV-cache sharing across requests (common in vLLM) is not directly available in Neuron Runtime — the Neuron adapter declares `SchedulerHints.kv_cache_shared = false`.

## Status

Both adapters are on the roadmap. The SageMaker wrapper is simpler and will likely land first. The Neuron Runtime direct adapter requires deeper integration work and benchmarking.
