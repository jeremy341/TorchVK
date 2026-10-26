# Architecture options

No architecture has been implemented yet. A direction will be chosen after the R0 competitor audit.

## Option A: independent PyTorch backend

```text
PyTorch model / autograd
        |
PyTorch dispatcher / PrivateUse1
        |
TorchVK C++ extension (storage, device, operators)
        |
Vulkan device / allocator / queues / pipeline cache
        |
GLSL or Slang -> SPIR-V compute kernels
        |
RX 5600 XT on Windows
```

Responsibilities: device and queue selection, tensor allocation and lifetime, CPU-GPU transfers, shape, stride and dtype handling, operator registration, command submission, synchronization, shaders, profiling, and explicit errors for unsupported operations.

Caveats:

- PrivateUse1 is a single extension slot. Several backends that each rename it can conflict in one process. Use separate test environments or processes.
- PyTorch's autograd formulas can be reused where they are registered. The backward pass still needs working operations and data that stays on the GPU. Autograd does not guarantee GPU training by itself.
- Research runs should use a strict GPU-only mode. It has to prove that backward, loss and the optimizer do not silently run on the CPU.
- Avoid a host synchronization and a transfer for every operation.
- C++20, CMake and GLSL/SPIR-V are initial suggestions. The allocator, shader language, Vulkan extension requirements and execution model all need evidence before they are chosen.

## Option B: upstream contribution

This should be seriously considered first. Work on [torchvulkan](https://github.com/zainsharief/torchvulkan) to fix RX 5600 XT support, add backward operations, instrument CPU fallback, or add low-VRAM allocation or kernels. This reuses their infrastructure and adds tests and benchmarks. A public research study and a useful upstream contribution are compatible.

## Option C: research tools

Build a Windows and RDNA1 benchmark suite, kernel experiments, a strict-backend profiler, or a compatibility database. These should work with existing implementations. This option may produce the most scientific value per hour.

## First low-level exercise

Write an independent Vulkan C++ test: select a device, allocate staging and device buffers, compile GLSL to SPIR-V, dispatch a vector add, synchronize, and compare the output with the CPU. After that, do GEMM. These are learning exercises, not a PyTorch training backend.

## Future Python usage (not functional)

```python
# Illustrative goal only
import torch
import torchvk

model = SmallMLP().to("torchvk")
loss = model(data.to("torchvk")).square().mean()
loss.backward()
optimizer.step()
```
