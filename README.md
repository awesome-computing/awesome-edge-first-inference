# Awesome Edge-First Inference [![Awesome](https://awesome.re/badge.svg)](https://awesome.re) [![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

> Run inference on-device or at the edge before round-tripping to a cloud API. Justify the network hop.

A curated, decision-guide awesome-list for choosing on-device / edge inference over a cloud API call — and for knowing honestly when you shouldn't.

This is a sibling in spirit to [awesome-cpu-first-ai](https://github.com/ranjithrajv/awesome-cpu-first-ai), but a different axis: that list is about CPU vs GPU as *hardware* for a given inference job. This list is about *where* the inference runs at all — on the device/at the edge vs a round trip to a cloud API — and the latency, privacy, offline-capability and cost tradeoffs that decision carries. The two questions are independent: you can run edge-first inference on a CPU, an NPU, or a small on-device GPU.

## Contents

- [Introduction](#introduction)
- [Decision Heuristics](#decision-heuristics)
- [On-Device Inference SDKs](#on-device-inference-sdks)
- [Browser / Edge Inference](#browser--edge-inference)
- [Edge Inference Serving](#edge-inference-serving)
- [Model Compression for Edge Deployment](#model-compression-for-edge-deployment)
- [Edge-First Product Patterns](#edge-first-product-patterns)
- [When You Actually Need the Cloud](#when-you-actually-need-the-cloud)
- [Further Reading](#further-reading)
- [Contributing](#contributing)
- [License](#license)

## Introduction

A large share of production inference workloads are not "ask a frontier model to reason about something hard." They're classification, embeddings, short-form transcription, simple vision tasks (object detection, OCR, face landmarking), and small-model chat/autocomplete. None of these categorically require a cloud round trip — modern phones, laptops, and even microcontrollers can run models sized for these tasks directly, and often the on-device version is the *better* engineering choice, not just the cheaper one:

- **Latency** — no network round trip means single-digit-millisecond inference instead of hundreds of milliseconds, which matters for anything interactive (keyboards, cameras, AR).
- **Privacy** — data never leaves the device, which simplifies compliance stories for health, biometric, and personal data.
- **Offline capability** — the feature keeps working on a plane, in a basement, or on a factory floor with no connectivity.
- **Cost at scale** — a cloud API call priced per request adds up; an on-device model is a fixed engineering cost that scales for free with your user base.

The tradeoff is real: on-device models are smaller, so they're categorically worse at open-ended reasoning, long-context tasks, and anything that benefits from a frontier model's scale. This list exists to help you tell those two situations apart, not to argue edge always wins.

**Licensing note:** every resource below is tagged with its license — `(MIT)`, `(Apache-2.0)`, `(BSD-3-Clause)`, `(open format)`, `(open standard)`, or `(proprietary)`. Where a category has both open-source and proprietary/vendor-locked options, the FOSS, cross-platform option is the default recommendation; the proprietary one is included because it's sometimes still the right call once you're already committed to that vendor's platform.

## Decision Heuristics

Lean **edge-first** when:
- The task is narrow and well-scoped (classify, embed, transcribe, detect, extract) rather than open-ended reasoning.
- Latency is user-perceptible and interactive (typing, camera, voice).
- The input data is sensitive (health, biometric, personal) and shipping it off-device adds compliance surface area.
- The feature must keep working offline or on flaky networks.
- You're serving enough volume that per-request API cost is a real line item.

Lean **cloud/frontier model** when:
- The task needs broad world knowledge, multi-step reasoning, or long-context synthesis that no edge-sized model currently matches.
- The workload is bursty/low-volume enough that a fixed on-device engineering investment doesn't pay for itself.
- You need to iterate on model quality weekly without shipping app updates.
- The device fleet is too heterogeneous or resource-constrained to guarantee a consistent on-device experience.

## On-Device Inference SDKs

FOSS, cross-platform first — reach for these before a single-vendor lock-in SDK:

- [TensorFlow Lite / LiteRT](https://ai.google.dev/edge/litert) (Apache-2.0) — Google's edge runtime for mobile, embedded, and IoT, the successor branding of TFLite.
- [ONNX Runtime Mobile](https://onnxruntime.ai/docs/tutorials/mobile/) (MIT) — ONNX Runtime's mobile-optimized build for iOS/Android with reduced binary size.
- [MediaPipe](https://github.com/google-ai-edge/mediapipe) (Apache-2.0) — Google's cross-platform pipeline framework for on-device vision, audio, and text tasks (face/hand landmarking, segmentation, LLM inference).
- [PyTorch ExecuTorch](https://github.com/pytorch/executorch) (BSD-3-Clause) — PyTorch's edge/mobile inference runtime, designed to run models exported from PyTorch with minimal overhead on-device.

Vendor-locked, proprietary — legitimate once you're already all-in on that platform, but not the FOSS default:

- [Core ML](https://developer.apple.com/documentation/coreml) (proprietary, Apple platforms only) — Apple's on-device inference framework across iOS/macOS, with the Neural Engine as a first-class target. Free to use as an Apple developer, but closed-source and locked to Apple hardware — reach for the FOSS options above unless you're already iOS/macOS-only.
- [Qualcomm AI Engine Direct SDK (QNN)](https://www.qualcomm.com/developer/software/qualcomm-ai-engine-direct-sdk) (proprietary) — vendor SDK for Snapdragon NPUs.

## Browser / Edge Inference

- [transformers.js](https://github.com/huggingface/transformers.js) (Apache-2.0) — run Hugging Face Transformers models directly in the browser via ONNX Runtime Web / WebGPU.
- [ONNX Runtime Web](https://onnxruntime.ai/docs/tutorials/web/) (MIT) — ONNX Runtime compiled to WebAssembly/WebGPU for in-browser inference.
- [WebNN](https://www.w3.org/TR/webnn/) (open standard, not software) — the emerging W3C Web Neural Network API for native-speed inference in the browser via the OS's ML stack. A spec, not a licensed codebase — implementations ship inside browsers.
- [ml5.js](https://ml5js.org/) (MIT) — friendly, beginner-oriented wrapper for in-browser ML, built on TensorFlow.js.

## Edge Inference Serving

- [NVIDIA Triton Inference Server](https://github.com/triton-inference-server/server) (BSD-3-Clause) — genuinely open source despite the NVIDIA name; supports edge/Jetson deployment targets alongside datacenter GPUs.
- [KServe](https://github.com/kserve/kserve) (Apache-2.0) — Kubernetes-native model serving that can be deployed at edge clusters, not just central cloud.
- [llama.cpp server](https://github.com/ggml-org/llama.cpp) (MIT) — lightweight local inference server, commonly used to stand up an edge/on-prem LLM endpoint.
- [LocalAI](https://github.com/mudler/LocalAI) (MIT) — OpenAI-API-compatible local inference server for running models on your own edge/on-prem hardware.

## Model Compression for Edge Deployment

- [llama.cpp / GGUF ecosystem](https://github.com/ggml-org/llama.cpp) (MIT; GGUF itself is an open format, not licensed software) — the quantization format and runtime that made running LLMs on consumer/edge hardware practical.
- [Hugging Face Optimum](https://github.com/huggingface/optimum) (Apache-2.0) — export and optimize (quantize, prune, graph-optimize) models for specific edge/accelerator targets.
- [Neural Network Distiller / general distillation techniques](https://github.com/IntelLabs/distiller) (Apache-2.0) — reference implementations for compressing models via distillation and pruning.
- [AIMET (AI Model Efficiency Toolkit)](https://github.com/quic/aimet) (BSD-3-Clause) — Qualcomm's open-source toolkit for quantization and compression targeting edge deployment.

## Edge-First Product Patterns

- On-device keyboards and predictive text (autocomplete running entirely on-device for latency and privacy).
- Camera-based real-time detection/segmentation (AR filters, document scanners) where cloud round-trip latency would break the interaction.
- Voice wake-word detection running locally, with only the post-wake-word audio (if anything) sent to a cloud model.
- Offline-first field applications (industrial, medical, agricultural) where connectivity can't be assumed.

## When You Actually Need the Cloud

Be honest about this list too:

- Open-ended reasoning, coding, or long-document synthesis tasks are still won decisively by frontier cloud models — no current edge-sized model is competitive.
- If you need to update model behavior weekly based on user feedback, shipping a new on-device model via app store review cycles is a real cost cloud APIs don't have.
- Heterogeneous device fleets (old phones, wildly different NPU/GPU capability) can make guaranteeing a consistent edge experience harder than it sounds — sometimes a cloud API is the more *reliable* choice, not just the lazier one.

## Further Reading

- [Apple: Deploying Transformers on the Apple Neural Engine](https://machinelearning.apple.com/research/neural-engine-transformers)
- [Google AI Edge documentation](https://ai.google.dev/edge)
- [W3C WebNN Explainer](https://webmachinelearning.github.io/webnn-status/)

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Please keep additions to real, actively-maintained projects, and prefer primary sources (official docs/repos) over blog posts when linking a tool.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](LICENSE)

To the extent possible under law, the contributors have waived all copyright and related or neighboring rights to this work. See [LICENSE](LICENSE).
