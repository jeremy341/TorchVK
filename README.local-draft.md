# TorchVK

PyTorch training over Vulkan Compute on consumer GPUs. Early research and prototyping.

Status: documentation and planning only. There is no complete PyTorch backend yet, and no performance claim is made. Benchmarks are published once they exist.

## Research question

Many consumer GPUs, especially older AMD cards, have real compute but cannot practically accelerate modern ML training. CUDA is NVIDIA-only. ROCm officially supports only some AMD hardware, and it is weakest on Windows. Vulkan Compute runs on almost all current AMD, Intel and NVIDIA GPUs, but it is rarely used for ML training.

TorchVK asks whether Vulkan Compute can give a practical path for PyTorch tensor execution and small training runs on hardware without a usable CUDA or ROCm path. The first goal is correctness and a minimal end-to-end training loop. Performance comparisons come later, once the implementation and evaluation protocol are stable.

## Intended usage

The target is a PyTorch device backend:

```python
device = torch.device("vulkan")

model = model.to(device)
x = x.to(device)

loss = model(x)
loss.backward()
optimizer.step()
```

Users should not need to know Vulkan or write shaders.

### Target architecture

```text
PyTorch / Python
      |
   TorchVK
      |
Vulkan Compute
      |
Consumer GPU
```

PyTorch's `PrivateUse1` out-of-tree backend mechanism is the leading option. The final design stays open until experiments show what works.

## Non-goals

- Not a CUDA replacement, and not an attempt to beat NVIDIA Tensor Cores at their own game
- Not a reimplementation of all of PyTorch
- Not support for every operator, GPU or operating system from the start
- Not a complete backend yet
- Not validated on every GPU, operator or OS
- Not making performance claims before reproducible benchmarks exist

The first objective is smaller: show that Vulkan can provide useful GPU-accelerated ML training on hardware without practical CUDA or ROCm support.

## Initial target

| Component | Hardware |
|---|---|
| Development GPU | AMD Radeon RX 5600 XT (6 GB VRAM, RDNA1) |
| OS | Windows |
| CPU baseline | AMD Ryzen 5 5600 |

Other vendors are a long-term goal, not an MVP requirement.

## Roadmap (short)

```text
Vulkan init -> compute shader -> GPU tensors -> GEMM -> MNIST -> CNN -> transformer experiments
```

The first milestone is training MNIST end to end on the RX 5600 XT through Vulkan, faster than the Ryzen 5 5600 CPU.

## Documentation

| Document | Contents |
|---|---|
| [PROJECT.md](PROJECT.md) | Engineering plan: scope, phases, effort, version roadmap, gates |
| [RESEARCH.md](RESEARCH.md) | Research question, hypothesis, related work, benchmarks, feasibility, paper plan |
| [HARDWARE.md](HARDWARE.md) | Why older AMD GPUs, and targets by generation |

---

This project is experimental. Nothing here is a benchmark result until it has been measured.
