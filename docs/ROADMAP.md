# TorchVK research and learning roadmap

This roadmap combines the project milestones with the learning each one needs. The current question is whether we can deliver a specific, measured benefit on older Windows AMD GPUs beyond what existing Vulkan training systems already offer. Start with R0. Its result may change the rest of the plan.

Planning pace: 8 to 12 focused hours per week. Calendar estimates are ranges. The learning path is in [LEARNING.md](LEARNING.md). Learn each topic shortly before the experiment that needs it, rather than finishing every course first.

## How to start

Do these in order:

1. Read [PROJECT_CHARTER.md](PROJECT_CHARTER.md), [RELATED_WORK.md](RELATED_WORK.md) and [DECISIONS.md](DECISIONS.md). Write down which statements are plans, which are reports, and which have not been checked on this PC. Begin the competitor review with the [torchvulkan repository](https://github.com/zainsharief/torchvulkan).
2. Record the machine and toolchain: GPU, driver, Vulkan device and API capabilities, compiler, CMake, Python and PyTorch versions. Save the exact output in a dated Markdown lab note.
3. Audit the closest existing PyTorch and Vulkan implementation: source revision, supported Windows path, native operators, fallback behavior, test coverage and documented training examples.
4. Attempt the smallest useful strict run the existing project supports, starting with MNIST if practical. Save commands, versions, successes and full errors. Add a larger workload only if it answers a specific open question.
5. Write up the gap and choose the next path: upstream contribution, research and benchmark tooling, or a narrow independent prototype. Do not start a full backend before this decision.

While R0 runs, begin the C essentials in [LEARNING.md](LEARNING.md). Auditing the current stack is part of learning it.

## Milestones

### R0: competitor and hardware feasibility (12 to 30 hours, weeks 1 to 3)

- [ ] Read the closest existing project's source, tests, supported operations, build instructions and fallback controls.
- [ ] Record the GPU, Windows driver, Vulkan runtime and device capabilities, and the relevant tool versions on the target machine.
- [ ] Reproduce an existing strict training path if the setup allows it. Start with MNIST. Keep every error and fallback observation.
- [ ] Run the available CPU and DirectML paths where they are compatible. Record precision and unsupported operations.
- [ ] Write a capability and gap report with exact revisions and evidence. Choose upstream work, research tooling, or an independent prototype.

Gate: find a concrete, measurable gap. If an existing implementation already meets the need, contribute to it or pick a research-tooling question instead. A failed setup can still be a useful result if it is reproducible and documented precisely.

Learning alongside R0: research-paper reading, Git and PowerShell, hardware and tool inventory, C syntax and pointers, Markdown lab notes, and citing sources.

### R1: correct Vulkan compute (15 to 35 hours, weeks 3 to 6)

- [ ] Build a minimal C++20 and CMake compute program that finds and selects the target device.
- [ ] Allocate and transfer buffers. Compile a small GLSL shader to SPIR-V.
- [ ] Dispatch an FP32 vector add, synchronize, read the result back, and compare it with a CPU reference.
- [ ] Cover empty, odd, small and large inputs. Document bounds and numerical tolerances.
- [ ] Measure upload, dispatch, readback and total time separately.

Gate: repeatable correct results and verified GPU execution on the target device. This proves one low-level compute path. A PyTorch backend needs separate evidence.

Learning alongside R1: modern C++ ownership and RAII, CMake, Vulkan device, buffer, descriptor, pipeline and command concepts, shader indexing, and the basic memory hierarchy. Start with the [C++ and CMake resources](https://isocpp.org/get-started) and the [Khronos compute tutorial](https://docs.vulkan.org/tutorial/latest/11_Compute_Shader.html). The full list is in [LEARNING.md](LEARNING.md).

### R2: GEMM and performance study (25 to 70 hours, weeks 5 to 10)

- [ ] Establish CPU correctness and optimized CPU library baselines.
- [ ] Implement a simple Vulkan matrix multiply. Cover rectangular and non-multiple dimensions.
- [ ] Add a tiled kernel only after the simple kernel is correct.
- [ ] Compare identical shapes at equal precision. Separate kernel-only time from transfers and end-to-end time.
- [ ] Record repeated samples, medians, spread, raw results and failure cases.

Gate: a reproducible correctness and performance study. A CPU win on small workloads, or a GPU plateau, is a valid measured result.

Learning alongside R2: matrix algebra, floating-point error, workgroup mapping, data reuse, memory bandwidth, occupancy, timing and basic statistics. Use [MIT Linear Algebra](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/), the [AMD RDNA Performance Guide](https://gpuopen.com/learn/rdna-performance-guide/), and the [evaluation protocol](EVALUATION.md).

### R3: choose the research path (after R0, roughly months 2 to 4)

Pick the smallest path that tests the gap:

- **Upstream contribution:** reproduce one failing, slow or missing case, then add a focused fix with evidence that meets upstream standards.
- **Research tooling:** build an operator and fallback tracer, a compatibility matrix, or a fair benchmark harness around existing backends.
- **Independent prototype:** only if R0 shows a reason the existing systems cannot answer the research question.

Gate: the scope is justified by the audit. Record the decision in [DECISIONS.md](DECISIONS.md).

Learning alongside R3: PyTorch tensors, autograd and dispatcher internals. Learn `PrivateUse1` only if the chosen path needs it. Follow the [PyTorch backend resources](LEARNING.md) and the [official PrivateUse1 tutorial](https://docs.pytorch.org/tutorials/advanced/privateuseone.html).

### R4: verified training path (roughly months 3 to 6, scope depends on R3)

- [ ] Support or measure only the operators the chosen experiment needs.
- [ ] Check forward, loss, backward and optimizer behavior against CPU references.
- [ ] Trace execution and show zero CPU fallback for any GPU-only claim.
- [ ] Run a small training workload. Record convergence, accuracy, transfers, latency and VRAM.
- [ ] Keep same-model, same-data, same-precision comparisons for any backend-specific claim.

Gate: the full training step is correct, reproducible, and its device residency is shown by evidence. A script that finishes is not enough.

Learning alongside R4: PyTorch tensors and autograd. Start with the [autograd quickstart](https://pytorch.org/tutorials/beginner/basics/autogradqs_tutorial.html) and apply the controls in [EVALUATION.md](EVALUATION.md).

### R5: one differentiated result (roughly months 5 to 10, 100 to 250 hours)

Pick one research contribution based on the evidence from R0 to R4. Options include strict fallback diagnostics, a useful low-VRAM training strategy, an RDNA1 or Windows compatibility finding, or an optimization with a controlled ablation.

- [ ] Define the hypothesis, metrics, comparison and failure conditions before tuning.
- [ ] Change one optimization at a time. Keep the raw data and configuration.
- [ ] Rerun the best result from a documented clean setup.
- [ ] Test a second GPU only if access is practical and it answers a portability question.

Gate: a measured improvement over a named baseline, or a useful, reproducible negative result. Do not claim novelty or compatibility beyond the evidence.

### R6: report, release and next decision (roughly months 6 to 12 or more, 60 to 130 hours)

- [ ] Freeze the code revision, environment, scripts, raw measurements and limitations.
- [ ] Write the report from the research record as it actually happened, not from the original hypothesis.
- [ ] Typeset the formal report in LaTeX when a school, competition or publication requires it. Keep the sources and the `.bib` file current.
- [ ] Rebuild the PDF from a clean directory. Check figures, equations, references, units and warnings.
- [ ] Ask a mentor or outside reviewer to challenge the method and the claims before release.

Gate: another person can follow the procedure, find the evidence for each claim, and reproduce the result or understand its limits.

Learning alongside R6: [Learn LaTeX](https://www.learnlatex.org/en/) and [BibLaTeX](https://ctan.org/pkg/biblatex). Keep the `.bib` file alongside the experiment evidence.

## Learning order

| When | Learn | Apply it to |
|---|---|---|
| Weeks 1 to 3 | research practice, source review, Git and PowerShell, C essentials | the R0 audit and dated experiment notes |
| Weeks 2 to 5 | C++20, ownership and RAII, debugging, CMake | CPU references and the R1 host program |
| Weeks 3 to 6 | CPU and GPU architecture, floating point, linear algebra basics | explaining data movement, indexing and bottlenecks |
| Weeks 5 to 9 | Vulkan compute, GLSL and SPIR-V | the R1 vector-add correctness proof |
| Weeks 8 to 12 | GEMM, numerical checks, timing, statistics and plots | the R2 naive and tiled study |
| After R0 | PyTorch dispatcher, autograd, PrivateUse1 | only the R3 path that the evidence selects |
| Start notes now; weeks 4 to 8 | research writing, references, LaTeX | the lab report, then the final report |

See [LEARNING.md](LEARNING.md) for topic checklists, exit checks and learning resources.

## Rough cumulative effort

| Outcome | Focused effort | Calendar at 8 to 12 hours/week |
|---|---:|---:|
| R0 audit and hardware and toolchain report | 12 to 30 hours | 1 to 3 weeks |
| Correct Vulkan vector add | 25 to 70 hours total | 1 to 2 months |
| Correct GEMM study | 60 to 140 hours total | 2 to 4 months |
| Useful verified PyTorch training experiment | 150 to 350 hours total | 4 to 8 months |
| Differentiated result and reviewed report | 300 to 650+ hours total | 6 to 12+ months |

These estimates overlap with learning time and depend on driver and toolchain problems. R0 can shorten the plan, redirect it or stop the implementation path. A narrower measured contribution can still be valuable. A broad, cross-GPU backend is a multi-year effort.
