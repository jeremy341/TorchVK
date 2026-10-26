# Related work

Snapshot date: 2026-10-10. This is a planning snapshot based on public code and documentation. No competitor has been independently reproduced on the RX 5600 XT yet. Check everything again before publishing a claim.

## Closest project: zainsharief/torchvulkan

- Repository: https://github.com/zainsharief/torchvulkan
- Package: https://pypi.org/project/torchvulkan/

What it does:

- Registers a PyTorch PrivateUse1 Vulkan device. Its shaders are written in Slang, compiled to SPIR-V, and built with a C++ extension and VMA allocation.
- Publishes cross-platform wheels, including Windows.
- Documents MNIST MLP training (forward, loss, backward and Adam) and a tiny GPT-style transformer.
- Falls back to the CPU for unsupported operations by default. The README says `TORCHVULKAN_STRICT=1` makes fallbacks raise errors. So strict runs need to be tested, not assumed to be native.
- Its `benchmark/benchmark.py` uses FP16 on the accelerator and FP32 on the CPU. That is a practical comparison, but it does not isolate backend speed at equal precision.

Our work has to go further than rerunning these examples. We need to check the native operator coverage and how the backward pass really behaves.

## Other relevant projects

**Peterc3-dev/torch-vulkan**: https://github.com/Peterc3-dev/torch-vulkan. An independent Vulkan PyTorch device aimed at LLM inference and GGUF kernels on a Radeon 890M. The README says full autograd training is not supported.

**PyTorch's Vulkan backend (legacy)**: https://docs.pytorch.org/tutorials/unstable/vulkan_workflow.html. Mainly for inference and mobile use, and now documented as unmaintained. This does not show that third-party Vulkan training does not exist.

**ExecuTorch Vulkan delegate**: https://docs.pytorch.org/executorch/stable/. A stack for exporting models and running edge inference. It is not general eager-mode PyTorch training.

**Meganeura**: https://arxiv.org/abs/2608.01563 and https://github.com/kvark/meganeura. Portable GPU training and inference with a graph compiler, autodiff and optimizers over Vulkan and Metal. Vulkan training is established in the research literature.

**torch-webgpu**: https://github.com/jmaczan/torch-webgpu and https://arxiv.org/abs/2604.02344. An adjacent out-of-tree backend that studies dispatch and fusion.

## Differentiation (hypotheses, not accomplishments)

| Direction | Evidence needed |
| --- | --- |
| RDNA1 and Windows compatibility | A successful strict GPU-only run, or a documented incompatibility and fix |
| Verified GPU-only training | An operator-level trace covering forward, backward, loss and optimizer, with zero fallback |
| Small-VRAM CNN training | Full CNN backward, and a fair comparison of supported model sizes, batch sizes and VRAM peaks |
| Architecture-specific speed | An ablation against upstream, on the same dtype, workload and hardware, with uncertainty reported |
| Better engineering diagnostics | Repeatable operator coverage tests and clear, actionable error messages |
| Reproducible research | Open scripts, raw logs, configuration manifests and stated limitations |

Warning: others may already have built some of these. Only measurements on current code versions can establish a real gap.

## Decision after R0

A. Contribute optimizations or driver fixes upstream. This is the preferred option if it is compatible.
B. Build a narrow research tool or plugin that works alongside an existing backend.
C. Write a separate backend only if a specific architecture, license or research limitation rules out A and B.

Do not describe the project as the "first Vulkan training backend" or the "only Vulkan PyTorch training" in the README, in outreach, or in a paper.
