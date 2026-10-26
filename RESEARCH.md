# TorchVK research plan

This file covers the research question, hypothesis, related work, experiment design and feasibility. The engineering plan is in [PROJECT.md](PROJECT.md), and the hardware case is in [HARDWARE.md](HARDWARE.md).

## Research question

A paper titled "I built a Vulkan ML framework" would describe software engineering, not research. The research question has to be testable:

> Can unsupported consumer GPUs provide practical machine-learning training performance through a vendor-neutral Vulkan backend?

Formal version:

> To what extent can a Vulkan-based PyTorch backend use the compute of unsupported consumer GPUs for neural network training, compared with the CPU and existing Windows GPU backends?

## Hypothesis

> A specialized Vulkan training backend can use compute that is currently unreachable on unsupported consumer GPUs. It can train neural networks significantly faster than the CPU while keeping model convergence equivalent.

This can turn out false. The main experimental GPU is the RX 5600 XT.

## Sub-questions

- RQ1: How much faster can Vulkan GPU training get than CPU training?
- RQ2: Which Vulkan kernel optimizations have the largest effect?
- RQ3: How does TorchVK compare with DirectML?
- RQ4: Does GPU execution keep numerical correctness and convergence?
- RQ5: How does performance vary across older GPU architectures?

## Related work

**Meganeura** was published on 3 August 2026, according to the project's notes. It reports portable GPU training and inference over Vulkan and Metal, including automatic differentiation, optimizers and memory planning. On a discrete AMD GPU, three of its training workloads ran faster than ROCm PyTorch. Its authors attribute the remaining gaps to kernel coverage, scheduling and arithmetic policy, not to Vulkan itself. These numbers have not been reproduced here.

The consequence is that "Vulkan can train neural networks" is already shown. We should not spend months repeating that work. Meganeura focuses on a portable train-to-deploy compiler and runtime.

**torch-webgpu** is a PyTorch `PrivateUse1` backend that studies dispatch overhead. It found that kernel fusion on Vulkan improved throughput substantially. That makes scheduling and fusion a key variable to test.

Our focus is different: PyTorch compatibility, unsupported commodity GPUs, Windows, accessibility, and a systematic study of kernel performance. These goals differ enough from the related work to justify the project, as long as the implementation and experiments are our own.

The question, reframed after Meganeura, is not "is Vulkan training possible?" It is: can we make Vulkan useful as a PyTorch training backend for hardware that the mainstream accelerator stacks have dropped?

## Experiment design

Benchmark the optimization process, not only the final version. Each stage is measured:

```text
Impl 0  CPU PyTorch
Impl 1  Naive Vulkan
Impl 2  Tiled GEMM
Impl 3  Optimized memory access
Impl 4  Kernel fusion
Impl 5  FP16 and specialized paths
```

**Benchmark 1, matrix multiplication.** Sizes 1024, 2048, 4096 and 8192 squared. Measure GFLOPS or TFLOPS, execution time and memory use. This shows how much of the RX 5600 XT's theoretical compute Vulkan can actually reach.

**Benchmark 2, MNIST MLP.** Not impressive alone, but a good correctness check. Compare PyTorch CPU with TorchVK on the GPU: epoch time, samples per second, final accuracy and loss curves. The model, data and optimizer stay the same. Only the backend changes.

**Benchmark 3, CIFAR-10 CNN.** Eventually ResNet-18. This adds convolution, normalization, gradients and optimizers, which is real training work.

**Benchmark 4, small transformer.** Models of 10M, 25M, 50M and 100M parameters. This shows the system is not only a Conv2D benchmark.

**Benchmark 5, backend comparison.** This is the central table of the paper:

| Backend | Hardware | Training speed | VRAM | Accuracy |
|---|---|---:|---:|---:|
| PyTorch CPU | Ryzen 5 5600 | baseline | system RAM | not yet measured for CNNs |
| PyTorch DirectML | RX 5600 XT | not yet measured | not yet measured | not yet measured for CNNs |
| TorchVK | RX 5600 XT | not yet measured | not yet measured | not yet measured for CNNs |

Other GPUs (RX 580, Vega 56, RX 5700 XT, RX 6600, Intel Arc) are optional extensions, not part of v1.

## Metrics beyond speed

**Energy.** Images trained per joule, or training time multiplied by average system power. A possible finding: GPU training may need less energy to reach the same accuracy, not only less time.

**Price and performance.** Used-market prices give context: RX 580 about 60 EUR, Vega 56 about 80 EUR, RX 5600 XT about 100 EUR, against new ML GPUs at 300 EUR or more. The metric is training samples per second per euro. The question is whether inexpensive second-hand gaming hardware can give useful ML acceleration.

## Planned contributions

A finished paper could claim four things:

1. TorchVK, an open-source PyTorch-compatible Vulkan backend for unsupported consumer GPUs.
2. GPU implementations of the operators needed to train real models.
3. An optimization study measuring how tiling, memory layout, fusion and precision affect Vulkan training.
4. A comparison of CPU, DirectML and Vulkan on CNNs and transformers, using commodity hardware.

## Feasibility estimates

These are rough guesses and should be revised against measurements.

| Goal | Estimate |
|---|---|
| Vulkan computes on the RX 5600 XT at all | about 99% |
| GPU tensors with simple ops (add, multiply, ReLU) | about 95% |
| Vulkan GEMM beats a naive CPU matmul | about 90% |
| Vulkan GEMM beats optimized CPU BLAS | 75 to 90% |
| Train MNIST completely on the GPU | about 85% |
| Train a CIFAR-10 CNN through the backend | 65 to 80% |
| Clearly beat PyTorch CPU on CNN training | 60 to 80% |
| Beat DirectML | 30 to 60%, an open research question |
| Match ROCm or CUDA in v1 | under 20%, not a success criterion |

Meganeura raises confidence in general: it shows automatic differentiation, optimizers, memory planning, Windows Vulkan support and useful AMD performance. Whether this project gets the hardware's advantage depends on software quality.

## Target paper

Working title: *TorchVK: Evaluating Vulkan as a Machine-Learning Training Backend for Unsupported Consumer GPUs*

This framing is built around the research question and RQ1 to RQ5.

## Jugend forscht

### Where the two projects stand

MIRA, an existing computer vision project, is the safer entry. It has dataset preparation, classification and detection, 19 documented experiments, repeatable benchmarks, about 90.6 mAP50, model export, TFLite deployment, webcam inference, a CLI, a dashboard and tests with CI.

TorchVK so far has an idea, a research direction, hardware choices and feasibility notes. It has no implementation, benchmarks or results yet.

If TorchVK works, its ceiling is higher. Strong CV projects are common, so MIRA's ceiling is about 8 or 9 out of 10. TorchVK spans GPU architecture, compute shaders, ML, PyTorch internals, performance work and open research. It also gives more to defend in the jury interview. The competition rewards a research question, experiments, and documented process, and TorchVK is built around that loop:

```text
Question -> Hypothesis -> Implementation -> Experiment -> Measurement
         -> Optimization -> New experiment -> Result -> Explanation
```

### Decision gates

Run both projects in parallel for a short time.

- Gate 1: a Python-to-Vulkan matmul on the RX 5600 XT gives a result close to the CPU result.
- Gate 2: Vulkan GEMM beats the optimized Ryzen 5 5600 CPU GEMM on large matrices.
- Gate 3: a small MNIST MLP trains end to end on the GPU (forward, backward, SGD), and its accuracy improves.

### Decision rules

| If TorchVK reaches | Then |
|---|---|
| Vulkan GEMM and only a few operators | Submit MIRA |
| Full MNIST training and the GPU beats the CPU | Competitive with MIRA |
| MNIST, a CIFAR-10 CNN, GPU faster than CPU, a DirectML comparison, an optimization study and an open-source repo | Submit TorchVK |
| All of the above, plus a small transformer, several GPUs and strong reproducible benchmarks | Submit TorchVK |

### Current call

Keep MIRA for now. TorchVK has the higher ceiling, but it should reach Gate 3 and show a measurable CPU speedup on CNN training before the submission changes. MIRA is still a good fallback. TorchVK is the riskier project with the larger payoff.
