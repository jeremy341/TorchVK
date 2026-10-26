# R0 competitor audit (2026-10-26)

Status: single-run feasibility evidence from one machine. There are no repeated timings, no same-precision benchmark and no VRAM study. The claims below cover only what was run here.

The machine record is in [R0_lab_note_machine_2026-10-26.md](R0_lab_note_machine_2026-10-26.md).

## Revisions and environments

- Upstream: `zainsharief/torchvulkan` at `078dd8ab0ed41aed41c6cbcd4def775b695fbd30` (2026-10-24). Source was read from a shallow clone in the session scratchpad.
- Installed package: `torchvulkan 0.2.1` from PyPI, with `torch 2.14.1+cpu`, `torchvision 0.29.1`, in a Python 3.11 venv. The PyPI wheel was not built from the audited commit, and the two were not compared file by file.
- DirectML baseline: `torch 2.4.1`, `torch-directml 0.2.5.dev240914`, `torchvision 0.19.1`, in a Python 3.11 venv.

## Upstream facts

These come from the source and README at the audited commit.

- It registers a `vulkan` device through PrivateUse1. About 737 `m.impl` registrations under `src/` cover forward and backward kernels, fused optimizer kernels (`_fused_adam_`, `_fused_adamw_`, `_fused_sgd_`) and FFT.
- By default, unsupported operations fall back to the CPU (`src/api/registration.cpp`). `TORCHVULKAN_STRICT` is enabled by any non-empty value other than `0`. Strict mode raises an error only on the aten fallback path.
- The README says `AdamW(fused=True)` runs its update on the CPU. The fused path is in `src/api/ops/optim.cpp`.
- The MNIST example uses `AdamW` on the default foreach path, not the fused path.

## Runs

| # | Test | Setup | Result | Source |
|---|---|---|---|---|
| 1 | Device detection and a small op | Python, `torchvulkan.is_available()`, computes `(a@b+1).exp().sum(0)` | Detected `AMD Radeon RX 5600 XT`, 6 GB total memory. Output matches the hand calculation (3035.5559 per element). | Inline, not saved to a file |
| 2 | Strict MLP train step, synthetic data | `TORCHVULKAN_STRICT=1`, 784-512-10 MLP, AdamW, batch 128, 3 steps | Loss 2.33, 1.91, 1.53. Parameters and gradients on `vulkan:0`. No errors. | Inline |
| 3 | Strict MNIST MLP, 3 epochs | `TORCHVULKAN_STRICT=1`, upstream `examples/mlp.py`, FP32 | 97.17% test accuracy. No fallback warnings in the log. | `scratchpad/mnist_strict/mlp_strict.log` |
| 4 | CPU baseline, same script | FP32, `mlp_cpu.py` (device forced to CPU) | 97.70% test accuracy | `scratchpad/mnist_strict/cpu.log` |
| 5 | DirectML baseline, same script | FP32, `mlp_dml.py` (device set to `torch_directml`) | 97.27% test accuracy. Wall time 65.4 s for one run, including data loading. Warning: `aten::lerp.Scalar_out` falls back to the CPU inside AdamW. | `scratchpad/mnist_strict/dml.log` |

Runs 3 to 5 are each a single run, and the example sets no seed. Accuracy differences of this size are not evidence that one backend is better. Wall times for runs 3 and 4 were not recorded.

## Negative and edge-case results

Strict mode was on unless noted.

| Operation | Result | Interpretation |
|---|---|---|
| `AdamW(fused=True)` step | Ran with no error | The README says it runs on the CPU. Strict mode did not flag it. |
| Complex elementwise multiply | Warning "Falling back to CPU", printed even with strict mode on | Strict mode did not turn this internal fallback into an error |
| bf16 matmul | Warning that it fell back to the CPU, also with strict on. bf16 is unsupported on this device (`is_bf16_supported() == False`). | Same gap as above |
| bf16 random generation and dtype copy | Warnings that they fell back to the CPU | Same gap as above |
| `torch.linalg.eig` (3x3) | Error: `Failed to create compute pipeline for binary_u8 (VkResult -3)` | The pipeline could not be created on this driver. This is not a CPU fallback. The cause is not isolated. |
| `torch.histc` | Error: `Failed to create compute pipeline for compare_u8 (VkResult -3)` | Same pattern. `VkResult -3` is `VK_ERROR_FEATURE_NOT_PRESENT` in the Vulkan headers. |
| `linalg.lstsq`, `linalg.qr`, `grid_sample`, forward and backward for conv2d, layer_norm and cross_entropy, fp16 matmul at 256, fp32 matmul at 1024, float64 matmul and exp | Ran with no error or warning | Only these shapes and inputs were tried. Results were not compared with a CPU reference. |

## What this shows and does not show

- As implemented, strict mode does not prove that a training step runs with zero CPU work. Three internal paths (complex arithmetic, bf16 matmul and dtype copy) printed CPU fallback warnings with strict mode on and did not raise errors. Fused AdamW ran without error. The README says it runs on the CPU, but that was not confirmed here.
- With strict mode on, the MNIST MLP run printed no fallback messages. That is evidence for this model and dtype only.
- Two u8 pipelines failed to build on this driver. That is a real compatibility gap on this machine. The cause is unverified.
- The DirectML run also falls back to the CPU for one AdamW operation (`lerp`). So no backend here keeps the optimizer fully on the GPU for this model.
- The three runs reached similar accuracy, and that is within the spread expected from unseeded runs. No speed conclusion is drawn. DirectML was timed once, and CPU and Vulkan were not timed.

## Not done

- Repeated timings, medians and variance, following the protocol in [EVALUATION.md](../EVALUATION.md).
- Vulkan vector-add and GEMM checks from R1 and R2.
- Numerical gradient comparison against the CPU for the Vulkan run.
- Driver and pipeline debugging for `binary_u8` and `compare_u8`.
- An upstream issue report or patch.

## Provisional decision

The evidence supports options B and C from [ARCHITECTURE.md](../ARCHITECTURE.md): contribute to the existing project, or build research tooling (a strict fallback tracer, a reproduction of the u8 pipeline failures). It does not support building an independent backend. The strict-mode gap and the RX 5600 XT pipeline failures are concrete starting points that can be reproduced. An independent backend is not justified by this audit alone. See [DECISIONS.md](../DECISIONS.md), entry D004.
