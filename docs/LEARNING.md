# TorchVK learning and reading roadmap

Read what you need for the **next experiment**, rather than mastering all Vulkan graphics.

## Foundations
- C++20 and memory ownership/RAII: https://www.learncpp.com/
- CMake: https://cmake.org/cmake/help/latest/guide/tutorial/
- PyTorch/autograd: https://pytorch.org/tutorials/beginner/basics/autogradqs_tutorial.html
- GPU workgroups, SIMD/SIMT, memory hierarchy: https://docs.nvidia.com/cuda/cuda-programming-guide/
- AMD GPUOpen architecture: https://gpuopen.com/

## Vulkan compute
- Khronos tutorial: https://docs.vulkan.org/tutorial/latest/00_Introduction.html
- Vulkan compute guide: https://docs.vulkan.org/guide/latest/compute_shaders.html
- Compute shader chapter: https://docs.vulkan.org/tutorial/latest/11_Compute_Shader.html
- Advanced Vulkan compute: https://docs.vulkan.org/tutorial/latest/Advanced_Vulkan_Compute/introduction.html
- LunarG Vulkan SDK: https://vulkan.lunarg.com/
- Learn buffers, device/queue, descriptors, pipelines, command submission, dispatch, synchronization, SPIR-V and GPU timestamps.

## PyTorch backend internals
- PrivateUse1: https://docs.pytorch.org/tutorials/advanced/privateuseone.html
- Dispatcher extensions: https://docs.pytorch.org/tutorials/advanced/extend_dispatcher.html

## Read these papers
1. Meganeura (portable Vulkan/Metal training): https://arxiv.org/abs/2608.01563
2. torch-webgpu / dispatch: https://arxiv.org/abs/2604.02344
3. OpenEye (accelerator architecture): https://arxiv.org/abs/2606.01450

Read: abstract -> introduction -> architecture -> results -> limitations. Note what is *already solved* and what remains.

## Learning milestones
- Explain CPU vs GPU, workgroup/invocation, memory hierarchy, GEMM/tiling.
- Run Vulkan vector add.
- Run naive and tiled GEMM with correctness checks.
- Understand where PyTorch dispatches and autograd interact with GPU ops.
- Audit existing implementations before designing an alternate backend.

Use this folder as Obsidian vault, keep Markdown in Git and local personal Obsidian configuration untracked.
