# Decisions and open questions

## 2026-10-10: D001, do not reinvent an existing Vulkan trainer

Complete the R0 competitor audit first. [zainsharief/torchvulkan](https://github.com/zainsharief/torchvulkan) already documents MNIST with Adam and a tiny GPT training path through the Vulkan PrivateUse1 backend. We need a more specific contribution.

## 2026-10-10: D002, strict GPU-only semantics

Any claim of GPU-only training requires documented zero CPU operator fallbacks across the whole training step, not only a successful model run.

## 2026-10-10: D003, target system

Provisional: Windows, RX 5600 XT (6 GB), Ryzen 5 5600 as the baseline, and Vulkan C++ and shader experiments. The architecture is still open.

## 2026-10-26: D004, R0 audit result (provisional)

Do not start an independent backend. Prefer contributing to [torchvulkan](https://github.com/zainsharief/torchvulkan), or building research tooling such as a strict-fallback tracer or a reproduction of the u8 pipeline failures.

Evidence is in [experiments/R0_COMPETITOR_AUDIT.md](experiments/R0_COMPETITOR_AUDIT.md). On the RX 5600 XT, the upstream package trained an MNIST MLP under `TORCHVULKAN_STRICT=1` to 97.17% accuracy with no fallback messages. Strict mode did not catch three internal CPU fallbacks. The `binary_u8` and `compare_u8` pipelines fail to build on this driver. These are single runs. No timing comparison has been made yet.

## Open questions

1. Does upstream torchvulkan install and run strictly on this Windows RDNA1 machine? Partly answered by D004: yes for MNIST, with the caveats above.
2. Where are the missing native operators, gradients, optimizer steps and driver issues?
3. Is an upstream contribution better than an independent backend?
4. What is actually new after reading Meganeura, torch-webgpu and the latest related work?
5. Which GPU kernel, allocator or scheduling bottleneck dominates on a 6 GB RDNA1 card?
6. Can a second GPU test reproducibility across GPU generations?
7. Which capabilities and extensions does the local AMD Vulkan driver expose?

## Next artifact

R0 is recorded in `docs/experiments/`. Any further claim needs its own experiment note, with commands, exact revisions, logs and both successful and failed results. Do not fabricate results.
