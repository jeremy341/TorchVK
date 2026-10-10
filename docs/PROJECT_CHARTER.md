# TorchVK project charter

## Mission
Investigate whether PyTorch training via Vulkan can be **practical, transparent and efficient on older consumer GPUs with poor mainstream CUDA/ROCm access**. Our reference experiment uses AMD RX 5600 XT 6 GB on Windows, compared with Ryzen 5 5600 CPU; these are targets, not results.

## Crucial novelty correction
As of 2026-10-10, [torchvulkan](https://github.com/zainsharief/torchvulkan) already documents:
- external PyTorch Vulkan device through PrivateUse1;
- MNIST training, autograd/backward and Adam;
- a small GPT-style Transformer example;
- Windows/Linux/macOS builds;
- native Vulkan operators plus CPU fallback for unsupported operations.

Therefore a basic Vulkan+PyTorch backend and MNIST training **cannot be sold as new**. We should test whether its reported training runs natively on the RX 5600 XT without fallback. We must also examine [torch-vulkan](https://github.com/Peterc3-dev/torch-vulkan), [Meganeura](https://arxiv.org/abs/2608.01563), official legacy PyTorch Vulkan and ExecuTorch.

## Primary research question
Can we deliver **fully traced Vulkan training** on older RDNA1 Windows GPUs with a measurable advantage in correct training throughput, memory use, operator coverage, reliability or reproducibility relative to existing approaches?

Candidate distinctions to *test*, not claim:
- Strict GPU-only backward and optimizer operations rather than hidden CPU fallback.
- Complete CNN backward / useful training beyond MNIST.
- Specific 6 GB VRAM strategies: activation workspace, tiling, batch size and caching.
- RDNA1/Windows driver issues and compatible optimized kernels.
- Fair reproduced cross-backend benchmark protocol, operator profiling and open compatibility matrix.

## Audience and value
Researchers optimizing GPU kernels; developers/student researchers with inexpensive unsupported AMD cards; users who want a known, working Windows PyTorch workflow.

## Project deliverables
1. Public, versioned feasibility/competitor audit and precise research question.
2. Working GPU compute experiments validated against CPU references.
3. A targeted solution chosen after the audit: upstream contribution, independent backend or benchmark/optimization layer.
4. Reproducible benchmark dataset, strict GPU-only instrumentation, logs and performance/accuracy analysis.
5. Conditional research report/paper on a measured technical contribution.

## Goals and non-goals
Goals: correct execution, actual GPU residency, measurements, replicability, documented limitations, useful open source.

Not v1 goals: universal hardware support, reinventing autograd, full 7B+ LLM training, replacing CUDA, maximal PyTorch operator coverage, or speedup promises.

## Acceptance gates
- **R0:** have a reason *not* to just use/contribute to an existing Vulkan-PyTorch package.
- **Compute proof:** correct Vulkan vector-add and GEMM on the target GPU, reproducible timing.
- **Training proof:** entire forward/backward/optimizer run passes strict no-CPU-fallback verification and reaches expected loss/accuracy.
- **Research proof:** a new experimentally demonstrated result relative to upstream alternatives; disclose null/negative results.

## Research practice
Never infer usable training performance from theoretical TFLOPS. Do not compare FP16 GPU against FP32 CPU when claiming isolated software-backend advantage. State driver, hardware, dtype, code revisions and error checks.
