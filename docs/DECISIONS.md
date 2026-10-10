# TorchVK decisions and questions

## 2026-10-10 — D001: don't reinvent an existing Vulkan trainer
**Decision:** complete R0 competitor audit first. Evidence: [zainsharief/torchvulkan](https://github.com/zainsharief/torchvulkan) already documents MNIST + Adam and a tiny GPT training path through Vulkan PrivateUse1. We need a more specific contribution.

## 2026-10-10 — D002: strict GPU-only semantics
**Decision:** all GPU-only training research requires documented zero CPU operator fallbacks through the whole training step, not merely successful model execution.

## 2026-10-10 — D003: target system
**Provisional:** Windows, RX 5600 XT (6 GB), Ryzen 5 5600 baseline, Vulkan C++/shader experiments. Architecture choice remains open.

## Critical questions
1. Will upstream torchvulkan install and run **strictly** on this Windows/RDNA1 machine?
2. Where are missing native operators, gradients, optimizer steps or driver issues?
3. Is an upstream contribution better than an independent backend?
4. What is truly novel after reading Meganeura, torch-webgpu and latest related work?
5. Which GPU kernel/allocator/scheduling bottleneck dominates on 6 GB RDNA1?
6. Can a second GPU test reproducibility across generations?
7. What capabilities/extensions does the local AMD Vulkan driver expose?

## Next recorded artifact
Write `docs/experiments/R0_COMPETITOR_AUDIT.md` only after actual inspection/testing, with commands, exact revisions, logs and failure/success evidence. Do not fabricate results.
