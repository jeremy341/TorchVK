# TorchVK milestone roadmap

Estimated with roughly 8–12 focused hours/week alongside school. These are **planning estimates**, not guarantees.

## R0 — FIRST: competitors and hardware feasibility (12–30h, weeks 1–3)
- [ ] Inspect [torchvulkan](https://github.com/zainsharief/torchvulkan): code, tests, native ops, CPU fallback, installation, Windows support.
- [ ] Record actual driver/device/Vulkan capabilities of RX 5600 XT.
- [ ] In a reproducible isolated environment, attempt strict GPU-only MNIST and a small Transformer step in upstream, with exact versions and error logs.
- [ ] Set up optimized PyTorch CPU and DirectML baselines, if compatible.
- [ ] Produce a capability and gap report and select: upstream contribution, benchmark/optimization layer or separate backend.

**Gate:** identify a concrete, measurable unmet need. If existing backend already solves it, contribute or pivot.

## R1 — Vulkan compute literacy (15–35h, weeks 3–6)
- [ ] Minimal C++20/CMake compute app and GPU discovery.
- [ ] FP32 vector add with CPU reference; large and odd shapes.
- [ ] Correct command buffers, staging, synchronization, validation.
- [ ] Separate transfer, kernel and total latency.

**Gate:** reproducible GPU execution on actual target, with correctness tests.

## R2 — GEMM and performance engineering (25–70h, weeks 5–10)
- [ ] CPU reference and optimized BLAS baseline.
- [ ] Naive Vulkan GEMM; check rectangular/nonmultiple shapes.
- [ ] Tiled GEMM, workgroup optimization and memory reuse.
- [ ] Record GFLOP/s and failure cases; compare GPU-resident and end-to-end timings.

**Gate:** correct performance study, even if CPU wins on small problems.

## R3 — training integration (40–100h, months 2–4)
Depending on R0 decision:
- [ ] Implement upstream missing operations/fixes and tests **or**
- [ ] create enough PrivateUse1-backed storage/transfer/elementwise/matmul operations for a training step **or**
- [ ] build scientifically valuable cross-backend profiler/benchmark harness.
- [ ] Validate autograd gradients and GPU-only operator tracing.

**Gate:** complete reproducible PyTorch training step with no hidden CPU fallback where claimed.

## R4 — MNIST and baseline study (40–110h, months 3–6)
- [ ] MLP forward/backward/loss/optimizer end-to-end.
- [ ] Stable convergence and validation accuracy.
- [ ] Compare CPU, DirectML, existing torchvulkan and our contribution under controlled precision and identical model/data.
- [ ] Collect kernel/transfer/dispatch times and 6GB VRAM use.

**Gate:** fully traced training; note MNIST alone is **not novel**.

## R5 — differentiated contribution (100–250h, months 5–10)
Select one based on measured gaps:
- [ ] CNN/CIFAR-10 with native Conv2D backward; or
- [ ] low-VRAM 6GB training strategy; or
- [ ] dispatch/fusion/tiling improvement for RDNA1; or
- [ ] robust GPU operator/fallback verification and reproducibility tooling.
- [ ] Perform controlled ablations and test on a second GPU if feasible.

**Gate:** documented technical improvement or useful negative result over current baseline.

## R6 — release / paper (60–130h, months 6–12+)
- [ ] Versioned environment and scripts, open raw measurements.
- [ ] Literature/related work and limitations.
- [ ] Publish credible OSS and research report/paper with mentor feedback.

## Rough cumulative horizon
| Outcome | Effort | Typical calendar |
| --- | ---: | --- |
| Audit + first working GPU compute | 25–70h | 1–2 months |
| Correct GEMM research proof | 60–140h | 2–4 months |
| Useful PyTorch training prototype | 150–350h | 4–8 months |
| Differentiated CNN/low-VRAM work + paper | 300–650+h | 6–12+ months |
| Mature cross-GPU framework | 1,000+h | multi-year |

**Do not implement everything upfront.** R0 can substantially change or shorten this roadmap.
