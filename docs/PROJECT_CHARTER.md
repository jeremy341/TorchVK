# TorchVK project charter

## Mission

Test whether PyTorch training through Vulkan can be practical, transparent and efficient on older consumer GPUs that have poor CUDA or ROCm support. The reference experiment uses an AMD RX 5600 XT with 6 GB of VRAM on Windows, compared with a Ryzen 5 5600 CPU. These are targets, not results.

## Correction on novelty

As of 2026-10-10, [torchvulkan](https://github.com/zainsharief/torchvulkan) already documents:

- an external PyTorch Vulkan device through PrivateUse1
- MNIST training with autograd, backward and Adam
- a small GPT-style transformer example
- Windows, Linux and macOS builds
- native Vulkan operators with a CPU fallback for unsupported operations

So a basic Vulkan PyTorch backend that trains on MNIST cannot be presented as new. The next step is to test whether torchvulkan's training runs natively on the RX 5600 XT without fallback. We should also examine [torch-vulkan](https://github.com/Peterc3-dev/torch-vulkan), [Meganeura](https://arxiv.org/abs/2608.01563), the official legacy PyTorch Vulkan backend and ExecuTorch.

## Primary research question

Can we deliver fully traced Vulkan training on older RDNA1 Windows GPUs, with a measurable advantage in correct training throughput, memory use, operator coverage, reliability or reproducibility compared with existing approaches?

Candidate distinctions. Each one must be tested before it is claimed:

- Strictly GPU-only backward and optimizer operations, with no hidden CPU fallback.
- Complete CNN backward pass and useful training beyond MNIST.
- Specific 6 GB VRAM strategies: activation workspace, tiling, batch size and caching.
- RDNA1 and Windows driver issues, and compatible optimized kernels.
- A fair, reproduced cross-backend benchmark protocol, operator profiling and an open compatibility matrix.

## Audience

Researchers who optimize GPU kernels. Developers and student researchers with inexpensive, unsupported AMD cards. Users who want a known, working Windows PyTorch workflow.

## Deliverables

1. A public, versioned feasibility and competitor audit with a precise research question.
2. Working GPU compute experiments checked against CPU references.
3. A targeted solution chosen after the audit: an upstream contribution, an independent backend, or a benchmark and optimization layer.
4. A reproducible benchmark dataset with strict GPU-only instrumentation, logs, and a performance and accuracy analysis.
5. A research report or paper, if the measured results support one.

## Goals and non-goals

Goals: correct execution, verified GPU residency, measurement, replicability, documented limitations, and useful open-source code.

Not goals for v1: universal hardware support, rebuilding autograd, training 7B+ language models, replacing CUDA, full PyTorch operator coverage, or speedup promises.

## Acceptance gates

- **R0:** a concrete reason not to simply use or contribute to an existing Vulkan PyTorch package.
- **Compute proof:** correct Vulkan vector add and GEMM on the target GPU, with reproducible timing.
- **Training proof:** a full forward, backward and optimizer run that passes strict no-fallback checks and reaches the expected loss or accuracy.
- **Research proof:** a new, experimentally shown result compared with upstream alternatives. Null and negative results are reported too.

## Research practice

Never estimate usable training speed from theoretical TFLOPS. Do not compare an FP16 GPU run with an FP32 CPU run when claiming a software backend advantage. Record the driver, hardware, data type, code revision and error checks for every run.
