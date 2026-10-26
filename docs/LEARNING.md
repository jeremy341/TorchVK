# TorchVK learning roadmap

This is a project-led learning path. Start the research now, and study each topic shortly before the experiment that needs it. At 8 to 12 focused hours per week, expect roughly 3 to 4 months to reach a sound Vulkan compute and GEMM study. That is a planning range, not a deadline.

## The order in brief

1. Start R0 and the research habits now. Neither needs Vulkan mastery first.
2. Learn enough C to read Vulkan's C API. Then use modern C++20 for the host program.
3. Learn CPU and GPU architecture alongside linear algebra and floating point.
4. Learn the Vulkan compute path and check one vector add against a CPU reference.
5. Study GEMM, measurement and numerical correctness before PyTorch backend internals.
6. Keep research notes and references from the start. Learn LaTeX before writing a formal report.

The host program is currently planned in C++20, CMake, Vulkan Compute and GLSL/SPIR-V. These are initial choices, and R0 may change them. Use C as a short bridge to Vulkan's handles, structs, pointers and function calls. The main host-side study should be C++20.

## Foundations (reading list)

- C++20 and ownership, RAII: https://www.learncpp.com/
- CMake: https://cmake.org/cmake/help/latest/guide/tutorial/
- PyTorch autograd: https://pytorch.org/tutorials/beginner/basics/autogradqs_tutorial.html
- GPU workgroups, SIMD and SIMT, memory hierarchy: https://docs.nvidia.com/cuda/cuda-programming-guide/
- AMD GPUOpen architecture material: https://gpuopen.com/

## Stage 0: research and tool fluency (weeks 1 to 2, alongside R0)

Read for this stage: the [project charter](PROJECT_CHARTER.md), the [related-work audit](RELATED_WORK.md) and the [evaluation protocol](EVALUATION.md). For the competitor review, start at the [torchvulkan repository](https://github.com/zainsharief/torchvulkan) and follow its current README, tests and examples.

### Learn

- Use Git branches, diffs, commits and tags to tie each result to its source revision.
- Use PowerShell and the Windows compiler and build tools to check versions, build programs and keep error output.
- Read technical papers in this order: abstract, problem, method, evaluation, limitations. Record the exact claim, the evidence, the hardware and software, and what was not tested.
- Keep apart a plan, a reported result, an independently reproduced result, an implementation and a measured result.
- Define a controlled comparison: same workload, inputs, precision, warmup, synchronization, repeats and recorded environment.

### Practice and exit check

- Write a dated Markdown lab note with the question, setup, command, result, failure and next step.
- Inventory the target PC: GPU model, driver, Vulkan device and API capabilities, compiler, CMake, Python and PyTorch versions. Save the exact command output.
- Be able to say what R0 has verified and what it has not. A failed installation or unsupported operation counts as an audit result when the steps and the error are recorded.

## Stage 1: C essentials (weeks 1 to 3, about 10 to 20 hours)

Study: [Microsoft's C language reference](https://learn.microsoft.com/en-us/cpp/c-language/c-language-reference?view=msvc-170). Use it as a reference while writing the three small CPU programs below.

Learn the parts needed to read a low-level API:

- Types, integer widths, floating-point values, expressions, control flow and functions.
- Arrays, structs, pointers, `const`, and pointer lifetime. Know the difference between pointers and references for later C++ work.
- Stack and heap, allocation and release, object layout, alignment and undefined behavior.
- Header files, compilation, linking, compiler diagnostics, and stepping through code in a debugger.

Practice with small CPU programs: vector addition, dot product and matrix multiplication. Check the results against small inputs worked by hand. Handle empty inputs and sizes that are not a multiple of the vector width. Learn these basics before advanced C features.

Exit check: read a Vulkan C function declaration and explain its pointer, struct, count and lifetime expectations. Compile and debug a small C program without copying a complete solution.

## Stage 2: modern C++20 and CMake (weeks 2 to 5, about 15 to 30 hours)

Study: [Standard C++: get started](https://isocpp.org/get-started), [Microsoft's guide to pointers and smart pointers](https://learn.microsoft.com/en-us/cpp/cpp/pointers-cpp?view=msvc-170), and the [CMake tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/).

Build on the C you already know rather than treating C++ as a separate set of syntax:

- Values, references, `const`, classes and structs, constructors and destructors, namespaces.
- RAII and ownership. `std::vector`, `std::array`, `std::span`, strings and smart pointers.
- Templates and the standard library at a practical level. Avoid advanced template metaprogramming.
- Separate interface from implementation, handle errors, and use a debugger and compiler warnings.
- CMake targets, include paths, linking and out-of-source builds.

Build a small C++ command-line program with CPU reference versions of vector add and matrix multiply, and add a CMake build. Use RAII for Vulkan objects later, so cleanup is correct on error paths.

Exit check: explain who owns each buffer and when it is released. Build the same program from a clean build directory.

## Stage 3: computer architecture, GPU execution and numerical basics (weeks 3 to 6, about 15 to 25 hours)

Study: [MIT OpenCourseWare Linear Algebra](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/), [AMD's RDNA architecture overview](https://gpuopen.com/rdna/), and its [RDNA Performance Guide](https://gpuopen.com/learn/rdna-performance-guide/). The [CUDA programming model](https://docs.nvidia.com/cuda/cuda-programming-guide/01-introduction/programming-model.html) is a useful general introduction to GPU execution. Check hardware-specific details against AMD's documentation.

Study only the concepts that help predict correctness or performance:

- Bits and bytes, integer representation, IEEE-754 floating point, rounding and numerical error.
- CPU cores, caches, RAM, memory bandwidth, SIMD, and why one large CPU core behaves differently from many GPU lanes.
- The GPU execution hierarchy: dispatch or grid, workgroups, invocations or threads, subgroups or wavefronts, and compute units.
- Registers, local and shared memory, VRAM, occupancy, latency hiding, memory coalescing, synchronization, races and atomics.
- Host-to-device transfer cost. Tell apart latency-bound, bandwidth-bound and compute-bound work.
- Matrix shapes, strides, row and column indexing, dot products, matrix multiplication, and why tiling reuses data.

Start with general concepts, then read the AMD RDNA material for the target GPU. CUDA documentation explains general GPU ideas, but its warp sizes, execution details and APIs are NVIDIA-specific. Check RDNA behavior against AMD's documentation, and query Vulkan device properties and subgroup capabilities rather than hard-coding assumptions.

Exit check: trace one input value from CPU memory into a GPU buffer, through a shader invocation, and back. Explain where transfer, memory access and arithmetic costs occur.

## Stage 4: Vulkan compute (weeks 5 to 9, about 20 to 40 hours)

Study: the [Khronos compute-shader tutorial](https://docs.vulkan.org/tutorial/latest/11_Compute_Shader.html), the [Vulkan compute guide](https://docs.vulkan.org/guide/latest/compute_shaders.html), and the [Vulkan SDK](https://vulkan.lunarg.com/). Follow the compute path and skip the graphics-only chapters.

Learn Vulkan in the order the first experiment needs it:

1. Loader, instance, physical device selection, logical device and compute queue.
2. Buffers, memory requirements, allocation, staging and upload, and readback.
3. GLSL compute shader, SPIR-V compilation, descriptor layouts and the compute pipeline.
4. Command pool and buffer, dispatch dimensions, fences, synchronization and validation output.
5. CPU reference comparison, boundary checks, odd sizes and error paths.
6. Timing that separates upload, kernel, readback and total elapsed time.

Exercise: dispatch an FP32 vector addition on the RX 5600 XT, read the result back, and compare it with the CPU reference for small, large, odd and empty inputs. Record the device, driver, compiler, shader, commit, tolerances and timings. Correctness and real device execution come before optimization. This establishes one low-level compute path. It does not establish a PyTorch backend.

Exit check: reproduce the same correct result from a clean build, and explain every synchronization and transfer on the path.

## Stage 5: linear algebra, PyTorch and GEMM (weeks 8 to 12, about 25 to 70 hours)

Study: return to [MIT Linear Algebra](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/), work through the [PyTorch autograd quickstart](https://pytorch.org/tutorials/beginner/basics/autogradqs_tutorial.html), and use the project's [evaluation protocol](EVALUATION.md) to shape measurements.

### Math and machine learning

- Linear algebra: matrix multiplication, shape rules, transposes, reductions and data layout.
- Calculus: derivatives, the chain rule, and gradients through a linear layer and a simple loss.
- Numerical computing: tolerances, accumulated error, an FP32 baseline and gradient checks.
- PyTorch: tensors, devices, autograd, loss, optimizer and the CPU reference path.
- Measurement: warmup, synchronization, repeated samples, medians and spread, seeded inputs, and plots made from raw data.

Use the CPU reference to understand correctness first. Then write a naive Vulkan GEMM and check rectangular and non-multiple dimensions. Only after that try a tiled GEMM and workgroup tuning. Report kernel-only and end-to-end times separately. The GPU can lose on small workloads, and that is a valid result.

Exit check: show that CPU and Vulkan outputs agree within a stated tolerance over a range of shapes. Reproduce the benchmark summaries from saved raw measurements.

## Stage 6: PyTorch backend internals (after the R0 decision)

Study if R0 selects this path: [PyTorch's PrivateUse1 backend guide](https://docs.pytorch.org/tutorials/advanced/privateuseone.html) and the [C++ dispatcher extension tutorial](https://docs.pytorch.org/tutorials/advanced/extend_dispatcher.html).

Learn these only if the evidence says a backend or an upstream contribution is the right path:

- The PyTorch dispatcher and operator registration.
- PrivateUse1, device and storage support, tensor layout and strides, and transfers.
- Autograd formulas and backward operators.
- Strict fallback detection through forward, loss, backward and the optimizer step.
- Extension build steps and version compatibility.

A tensor placed on a device, and a training script that completes, are not enough to show full GPU training. Record which operations ran natively, and show zero CPU fallback for any GPU-only claim. If R0 favors an upstream contribution or a benchmark tool, focus the learning on that path.

## Stage 7: research writing, LaTeX and references (start notes now, learn LaTeX around weeks 4 to 8)

Study: [Learn LaTeX](https://www.learnlatex.org/en/) for the document workflow, and [CTAN's BibLaTeX reference](https://ctan.org/pkg/biblatex) for citations and bibliographies.

Keep the live research notebook in Markdown from the start. That makes experiment logs easy to write while the research question is still changing. LaTeX becomes worth the effort when the report needs equations, citations, cross-references, tables and a consistent PDF layout.

### Learn in this order

1. The beginner lessons on document structure, sections, figures, tables, cross-references, mathematics, citations and error handling.
2. Write a short lab report with one equation, one plot, one results table and references to the actual sources.
3. Learn a `.bib` file and the basics of biblatex and Biber. Keep one source record, and cite each claim close to where it is made.
4. Use the required competition, school or publication template when one exists. Keep custom formatting to a minimum.
5. Compile the same report cleanly from an empty folder. Check the PDF for broken references, unreadable plots, missing units and citation errors.

### What the final report should explain

- The research question, and why existing work leaves a specific question open.
- The hardware and software environment, and the exact experiment setup.
- Correctness checks, fallback checks, controls and the measurement method.
- Results with units, variability, limitations, and failed or negative experiments.
- Reproduction steps, the location of raw data, the code revision, and references.

Sound method is the basis of the report. Keep the Markdown notes and the raw CSV or JSON measurements next to the LaTeX source.

## Checkpoints

| Checkpoint | You should be able to | Project artifact |
|---|---|---|
| R0 complete | Inspect a competitor, the target hardware and their limitations without claiming more than the evidence shows | Competitor and hardware audit with exact logs |
| C and C++ ready | Compile, debug and explain CPU reference code and resource ownership | Small C and C++ reference programs and a CMake build |
| Vulkan compute ready | Dispatch and check a GPU vector add | A reproducible vector-add experiment |
| GEMM ready | Validate matrix shapes and compare naive and tiled kernels fairly | A correctness matrix and raw benchmark data |
| Backend ready | Explain dispatch, autograd, storage and fallback evidence | Only the implementation or study selected after R0 |
| Report ready | Build a referenced PDF from recorded data and explain its limitations | LaTeX report, `.bib` file and reproducibility files |

## Resource map

Each study link appears in the stage where it is used. Project references: [research charter](PROJECT_CHARTER.md), [related work](RELATED_WORK.md), [architecture options](ARCHITECTURE.md), [evaluation protocol](EVALUATION.md) and [open decisions](DECISIONS.md).

## Earlier reading list (restored)

These resources and milestones were in an earlier version of this file. They are kept here as they were.

### Vulkan compute

- Khronos tutorial: https://docs.vulkan.org/tutorial/latest/00_Introduction.html
- Vulkan compute guide: https://docs.vulkan.org/guide/latest/compute_shaders.html
- Compute shader chapter: https://docs.vulkan.org/tutorial/latest/11_Compute_Shader.html
- Advanced Vulkan compute: https://docs.vulkan.org/tutorial/latest/Advanced_Vulkan_Compute/introduction.html
- LunarG Vulkan SDK: https://vulkan.lunarg.com/
- Topics to learn: buffers, device and queue, descriptors, pipelines, command submission, dispatch, synchronization, SPIR-V and GPU timestamps.

### PyTorch backend internals

- PrivateUse1: https://docs.pytorch.org/tutorials/advanced/privateuseone.html
- Dispatcher extensions: https://docs.pytorch.org/tutorials/advanced/extend_dispatcher.html

### Papers to read

1. Meganeura (portable Vulkan and Metal training): https://arxiv.org/abs/2608.01563
2. torch-webgpu and dispatch: https://arxiv.org/abs/2604.02344
3. OpenEye (accelerator architecture): https://arxiv.org/abs/2606.01450

Read each paper as: abstract, introduction, architecture, results, limitations. Note what is already solved and what is still open.

### Learning milestones

- Explain CPU versus GPU, workgroups and invocations, the memory hierarchy, and GEMM tiling.
- Run a Vulkan vector add.
- Run naive and tiled GEMM with correctness checks.
- Understand where PyTorch dispatches, and how autograd interacts with GPU operations.
- Audit existing implementations before designing an alternative backend.

Use this folder as an Obsidian vault. Keep the Markdown in Git, and keep personal Obsidian configuration untracked.
