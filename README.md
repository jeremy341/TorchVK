# TorchVK

Research into PyTorch training through Vulkan on older consumer GPUs.

TorchVK asks whether older Windows AMD GPUs, starting with the Radeon RX 5600 XT (6 GB), can run verified and efficient ML training through Vulkan. AMD's ROCm support for these cards is limited. The project is at the planning and feasibility stage. It does not yet provide a usable backend.

Existing projects already document PyTorch training over Vulkan, including MNIST and a small transformer. For example, [torchvulkan](https://github.com/zainsharief/torchvulkan) does this. TorchVK does not claim to be the first Vulkan training backend. The first step is to find and measure a specific gap: hidden CPU fallback, compatibility on older hardware, low-VRAM operation, or speed. Then decide whether contributing upstream makes more sense than writing a separate implementation.

## Start here

- [Documentation and research index](docs/INDEX.md)
- [Project charter](docs/PROJECT_CHARTER.md)
- [Related work](docs/RELATED_WORK.md)
- [Research roadmap](docs/ROADMAP.md)
- [Benchmark protocol](docs/EVALUATION.md)
- [Contributor instructions](AGENTS.md)

## Possible architecture (not implemented)

```text
PyTorch -> external backend (PrivateUse1) -> Vulkan runtime -> SPIR-V compute shaders -> GPU
```

The design is still open. Planned investigations include GPU compute correctness, matrix multiplication and tiling, strict GPU-only training, and controlled comparisons with CPU, DirectML and Vulkan backends.

Status: research and documentation only. No performance or compatibility results are claimed.
