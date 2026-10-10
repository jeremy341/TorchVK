# TorchVK architecture options

**No architecture has yet been implemented.** Choose a direction after R0 competitor audit.

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

Responsibilities: device/queue selection, tensor allocation & lifetime, CPU-GPU transfers, shape/stride/dtype handling, operator registration, command submission, synchronization, shaders, profiling, explicit errors for unsupported operations.

Important caveats:
- PrivateUse1 is a single extensibility dispatch slot; multiple backends that rename it may conflict in one process. Separate test environments/processes.
- PyTorch autograd formulas are reusable where registered, but backward requires working operations and GPU residence; autograd is not a free guarantee of GPU training.
- Use a **strict GPU-only mode** in research, with proof that backward, loss and optimizer are not silently running on CPU.
- Avoid one host synchronization and transfer per operation.
- C++20, CMake and GLSL/SPIR-V are initial suggestions. Need evidence before selecting allocator, shader language, Vulkan extension requirements, execution model.

## Option B: upstream contribution (seriously consider first)
Work on [torchvulkan](https://github.com/zainsharief/torchvulkan) to fix RX 5600 XT, backward support, CPU-fallback instrumentation, low-VRAM allocation or kernels. Reuse their infrastructure and provide tests/benchmarks. A public research study and independently useful upstream contribution are compatible.

## Option C: research tools
Build a Windows/RDNA1 benchmark suite, kernel experiments, strict-backend profiler and compatibility database that work with existing implementations. This may deliver the most scientific value per hour.

## Low-level first exercise
Independent Vulkan C++ test: select device -> allocate staging/device buffers -> compile GLSL to SPIR-V -> dispatch vector add -> synchronize -> compare CPU output. Then GEMM. Do not confuse these learning experiments with a useful PyTorch training backend.

## Tentative future Python usage (NOT FUNCTIONAL)
```python
# Illustrative goal only
import torch
import torchvk
model = SmallMLP().to("torchvk")
loss = model(data.to("torchvk")).square().mean()
loss.backward()
optimizer.step()
```
