# TorchVK: Codex instructions

Read `docs/INDEX.md` and `docs/RELATED_WORK.md` before making changes. This is an early-stage research project, **not an already working Vulkan training backend**.

## Goal
Investigate whether Vulkan can provide **strictly verified, efficient PyTorch training on older Windows AMD GPUs with limited ROCm support**, initially the RX 5600 XT (6 GB), compared to optimized CPU, DirectML, and existing Vulkan implementations.

**IMPORTANT:** `zainsharief/torchvulkan` already documents PyTorch training with MNIST, Adam and a tiny Transformer. We cannot claim Vulkan + PyTorch training is novel. Read the related-work notes and prove a specific research gap before building a duplicate.

## Mandatory read order
1. `docs/PROJECT_CHARTER.md`
2. `docs/RELATED_WORK.md`
3. `docs/ROADMAP.md`
4. `docs/ARCHITECTURE.md`
5. `docs/EVALUATION.md`
6. `docs/DECISIONS.md`

## Rules for coding agents
- Inspect actual repository state before planning. Do not assume plans are implemented.
- FIRST milestone is **R0: audit current alternatives and validate RX 5600 XT behavior**, not writing a full backend.
- Compare contributing to existing backends versus independently implementing one. Do not assume a standalone implementation is the right path.
- State acceptance criteria and planned edits before implementation. Prefer small, testable changes.
- Always separate **planned, reported, independently reproduced, implemented, measured**.
- Never claim "GPU-only" if an operation in forward, backward, loss or optimizer silently falls back to CPU.
- For benchmarks record hardware, driver, PyTorch version, git commit, dtype, shape, warmup, synchronization, samples and failure modes. Match precision and configuration for isolating backend performance.
- Test computations with a CPU correctness reference including edge cases and numerical tolerances.
- Avoid indiscriminate dependencies or scaffolding. C++20, CMake, Vulkan Compute, GLSL/SPIR-V and later Python/PyTorch are working defaults, not architectural commitments.
- Document negative experimental results; no fabricated numbers, novelty, compatibility or GPU support.
- Respect upstream licenses and attribution. Do not copy existing projects' implementations without approval.
- Keep experiment logs and a dated decision record.
- Do not perform unrelated changes or major research-direction pivots without explaining tradeoffs.

## Next action
Complete `R0` from the roadmap, write a reproducible competitor audit, and determine whether TorchVK should be an upstream contribution, research harness, or independent backend.
