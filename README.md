# TorchVK

**Experimental research into efficient PyTorch training through Vulkan on older consumer GPUs.**

TorchVK investigates whether older Windows AMD GPUs with limited practical ROCm support (starting with Radeon RX 5600 XT, 6 GB) can run verified, efficient ML training with Vulkan. The project is in the **planning and feasibility stage**. It does not yet offer a usable backend.

**Important:** Existing projects such as [torchvulkan](https://github.com/zainsharief/torchvulkan) already document PyTorch/Vulkan training, including MNIST and small-Transformer examples. TorchVK does **not** claim to be the first Vulkan training backend. Our initial task is to identify and measure specific gaps in older-GPU compatibility, hidden CPU fallback, low-VRAM execution, or performance, and determine whether contributing upstream is preferable to a separate implementation.

## Start here
- [Documentation and research index](docs/INDEX.md)
- [Project charter](docs/PROJECT_CHARTER.md)
- [Related work and differentiation](docs/RELATED_WORK.md)
- [Research roadmap](docs/ROADMAP.md)
- [Benchmark protocol](docs/EVALUATION.md)
- [Codex / contributor instructions](AGENTS.md)

## Possible architecture (not implemented)
```text
PyTorch → external backend / PrivateUse1 → Vulkan runtime → SPIR-V compute shaders → GPU
```

The approach remains open pending a competitor and hardware feasibility audit. Planned investigations include GPU compute correctness, GEMM/tiling, strict GPU-only training, and controlled CPU/DirectML/Vulkan comparisons.

**Status: research and documentation only. No performance or compatibility results are claimed.**
