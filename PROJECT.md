# TorchVK project plan

This file covers scope, architecture, phases, effort and gates. The research questions are in [RESEARCH.md](RESEARCH.md), and the hardware case is in [HARDWARE.md](HARDWARE.md).

## Goal

Build a PyTorch-compatible Vulkan training backend for older or unsupported consumer GPUs, starting with the RX 5600 XT on Windows.

This is not meant to be another full ML framework. TorchVK would provide the GPU execution backend. Users keep writing ordinary PyTorch and do not need to know Vulkan.

The estimate for a solid first public version is 150 to 300 hours.

## Name

The chosen name is **TorchVK** for the backend and runtime. The working title is "TorchVK: Vulkan-Accelerated Machine Learning Training on Unsupported Consumer GPUs."

Names considered and dropped: Voltaic, VulkTorch, OpenTrainVK, ForgeVK, Nerva, VulcanML (confusable with Vulkan) and VulkanTrain (accurate but generic).

## Responsibilities

- Detect and select Vulkan GPUs
- Allocate GPU memory and reuse it
- Move tensors between CPU and GPU
- Run tensor operations: matmul, convolution, activations, normalization, softmax
- Backward operations and gradients
- Optimizer updates
- Synchronization
- Shader compilation and caching
- Later: FP16, kernel fusion, transformer-specific kernels

## Target user code

```python
device = torch.device("vulkan")

model = model.to(device)
x = x.to(device)

loss = model(x)
loss.backward()
optimizer.step()
```

## Architecture direction

```text
PyTorch
   |
TorchVK (PrivateUse1 backend)
   |
Vulkan runtime
   |
SPIR-V compute kernels
   |
RX 5600 XT
```

PyTorch supports external accelerator backends through `PrivateUse1`, so an out-of-tree backend is a valid design. It is the leading candidate, but it is not decided. Phase 0 is meant to test it.

## Scope limits

- Windows and the RX 5600 XT only, at first. Other vendors and operating systems come later.
- v1 is deliberately small. See the version roadmap below.
- No extra dependencies, no optimization before correctness, and no claims without measurements.

## Development approach

- Correctness first. Each operator gets a CPU reference, then a Vulkan implementation, then an output comparison across tensor sizes. Benchmarks come last.
- Small, measurable milestones are preferred over large abstractions.
- Architecture decisions are written down.

## Phases

**Phase 0: research and architecture (10 to 20 h).** Study the Vulkan compute pipeline, the PyTorch `PrivateUse1` mechanism, tensor representation, shader compilation, allocator design and benchmark setup. Output: a validated architecture diagram.

**Phase 1: Vulkan compute proof of concept (15 to 30 h).** GPU detection, instance and device setup, buffers, upload and download, a compute shader and element-wise add. Goal: compute `C = A + B` entirely on the RX 5600 XT.

**Phase 2: GPU tensor runtime (20 to 40 h).** Shape, dtype and strides; GPU allocation; copies; buffer reuse; synchronization. This is the foundation for everything after.

**Phase 3: GEMM (25 to 60 h).** The first serious performance work: naive matmul, then tiling, shared or local memory, vectorized loads and workgroup tuning. Goal: beat CPU BLAS on large matrices. This phase could become a large part of the paper.

**Phase 4: basic neural network training (30 to 50 h).** ReLU, linear layer, MSE and cross-entropy loss, backward pass, SGD. Goal: MNIST entirely on the GPU. This is the first major milestone.

**Phase 5: CNN support (30 to 60 h).** Conv2D with backward pass, pooling, normalization and better optimizers. Goal: CIFAR-10. At this point the backend starts to be useful for real work.

**Phase 6: benchmarking and research (20 to 40 h).** CPU against DirectML against TorchVK, covering epoch time, samples per second, VRAM, accuracy, power, kernel timings and optimization ablations.

**Phase 7: polish and open source (15 to 30 h).** API cleanup, examples, installer, CI, tests, a GPU compatibility matrix and a contribution guide.

## Effort tiers

| Tier | Hours | Contents |
|---|---|---|
| Working proof of concept | 50 to 80 | Vulkan compute, tensor ops, basic GEMM, possibly MLP training |
| Proper MVP | 100 to 180 | GPU tensors, optimized GEMM, gradients, MNIST, basic PyTorch integration |
| Strong research version | 180 to 300 | MNIST, CIFAR-10, GPU faster than CPU, DirectML comparison, optimization study, reproducible benchmarks, open-source repo |
| Mature alternative backend | 500+ | Broad operator coverage, transformers, FP16, multiple GPUs, strong PyTorch compatibility |

The first paper does not need 500 hours.

## Version roadmap

**v0.1:** GPU detection, CPU-GPU tensor copies, FP32, add, subtract, multiply, ReLU, matmul, linear layer, MSE, softmax, cross-entropy, backward pass, SGD and an MNIST example.

Success criterion: train an MLP on MNIST entirely on the RX 5600 XT through Vulkan, and do it faster than the Ryzen 5 5600 CPU.

**v0.2:** Conv2D, max pooling, batch norm, Adam and AdamW. Goal: a CIFAR-10 CNN.

**v0.3:** LayerNorm, RMSNorm, RoPE, attention and FP16. Goal: a small transformer.

### Long-term

```text
TorchVK
  Training   : CNNs, transformers, LoRA, optimizers
  Inference  : FP32, FP16, quantization
  GPUs       : AMD, Intel, NVIDIA
  Platforms  : Windows, Linux
```

Inference is a lower priority. llama.cpp already does Vulkan inference well. Training is the less-explored gap.

## Success gates

| Level | Criterion |
|---|---|
| Minimum | Vulkan matrix multiplication is correct on the RX 5600 XT |
| Promising | Vulkan GEMM beats the Ryzen 5 5600 on large workloads |
| Solid | Full MNIST training on the GPU |
| Strong | CIFAR-10 CNN training, a clear CPU speedup, a DirectML comparison and reproducible benchmarks |
| Exceptional | CNN, small transformer, several GPUs, an optimization study, an open-source release and a paper |

## Public statement

> TorchVK aims to make GPU-accelerated neural network training available on AMD GPUs without practical CUDA or ROCm support, by turning PyTorch workloads into Vulkan compute kernels.

The first target is narrow: MNIST on the RX 5600 XT, faster than the Ryzen 5 5600. If that works, the project is worth continuing.
