# Related work and differentiation (2026-10-10)

This is a **planning snapshot** based on public code/docs. No competitor run has been independently reproduced on our RX 5600 XT. Re-verify before publishing a claim.

## Closest: zainsharief/torchvulkan
https://github.com/zainsharief/torchvulkan
https://pypi.org/project/torchvulkan/

- PyTorch PrivateUse1 Vulkan device, Slang shaders compiled to SPIR-V, C++ extension and VMA allocation.
- Reports cross-platform wheels including Windows.
- Documents MNIST MLP training (forward, loss, backward, Adam) and a tiny GPT/Transformer example.
- Unsupported operations fall back to CPU by default. `TORCHVULKAN_STRICT=1` is documented to error on fallback. Therefore it is critical to test strict runs, not assume training is native.
- Published `benchmark/benchmark.py` uses FP16 for accelerator and FP32 for CPU. Useful practical comparison; **not a same-dtype comparison isolating backend quality**.
- Our role must be more than reproducing these examples. Inspect actual native operation coverage and backward behavior.

## Other relevant efforts
**Peterc3-dev/torch-vulkan** — https://github.com/Peterc3-dev/torch-vulkan  
Independent Vulkan/PyTorch device, oriented to LLM inference and GGUF kernels on Radeon 890M; its README says full autograd training is not supported.

**PyTorch historical Vulkan backend** — https://docs.pytorch.org/tutorials/unstable/vulkan_workflow.html  
Primarily inference/mobile-oriented and now documented as unmaintained. NOT evidence that third-party Vulkan training does not exist.

**ExecuTorch Vulkan delegate** — https://docs.pytorch.org/executorch/stable/  
A model export/edge inference/deployment stack rather than general eager PyTorch training.

**Meganeura** — https://arxiv.org/abs/2608.01563 ; https://github.com/kvark/meganeura  
Portable GPU training and inference with graph compiler, autodiff and optimizers over Vulkan/Metal. Training on Vulkan is established in research.

**torch-webgpu** — https://github.com/jmaczan/torch-webgpu ; https://arxiv.org/abs/2604.02344  
Adjacent out-of-tree backend and dispatch/fusion research.

## Differentiation table (hypotheses, NOT existing accomplishments)

| Direction | Evidence needed |
| --- | --- |
| RDNA1/Windows compatibility | Real successful strict GPU-only run or documented incompatibility and fix |
| Verified GPU-only training | Operator-level trace including forward/backward/loss/optimizer and zero fallback |
| Small-VRAM CNN training | Full CNN backward and fair comparison of model/batch support and VRAM peaks |
| Architecture-specific speed | Ablation versus upstream, same dtype/workload/hardware with uncertainty |
| Better engineering diagnostics | Repeatable operator coverage tests and documented actionable errors |
| Reproducible research | Open scripts, raw logs, configuration manifests and explicit limitations |

**Warning:** Others may already implement some of these. Only measurements on current revisions determine a valid gap.

## Practical decision after R0
A. Contribute optimizations/driver fixes upstream (default preferred if compatible).
B. Build a narrow research instrument/plugin alongside an existing backend.
C. Write a separate backend only if a specific architecture/license/research limitation makes A or B inappropriate.

Do not claim “first Vulkan training backend” or “only Vulkan PyTorch training” in README, outreach or paper.
