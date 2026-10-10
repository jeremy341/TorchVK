# TorchVK research and Codex docs

This `docs/` directory can also serve as the Obsidian vault. All Markdown is tracked in Git; personal Obsidian settings may be excluded.

- [PROJECT_CHARTER.md](PROJECT_CHARTER.md) — objective, scope, contribution, feasibility criteria
- [RELATED_WORK.md](RELATED_WORK.md) — exact alternatives and novelty risks
- [ARCHITECTURE.md](ARCHITECTURE.md) — candidate PyTorch/Vulkan system layers and alternatives
- [ROADMAP.md](ROADMAP.md) — gated milestones and approximate time
- [EVALUATION.md](EVALUATION.md) — reproducible benchmarks and scientific claims
- [DECISIONS.md](DECISIONS.md) — open questions and decisions
- [LEARNING.md](LEARNING.md) — study resources and research papers
- [EXPERIMENT_TEMPLATE.md](EXPERIMENT_TEMPLATE.md) — reusable experiment note

**Status on 2026-10-10:** research/planning. The original repo had only a 9-byte README heading and LICENSE; verify latest state before beginning tasks.

## Glossary
- **Backend:** platform-specific implementation of ML tensor operations.
- **PrivateUse1:** PyTorch extension dispatch mechanism for custom accelerator backends.
- **GEMM:** general matrix multiplication.
- **SPIR-V:** intermediate binary representation of GPU compute shaders.
- **Autograd:** PyTorch automatic gradient computation.
- **CPU fallback:** unsupported device operation gets executed on CPU, possibly hiding missing GPU support.
- **Ablation:** controlled test that changes or removes one optimization.

## Research stance
The question is *not* "Can PyTorch train using Vulkan?" since existing packages already document that capability. It is what can be improved **on older AMD hardware under Windows**, with strict verification, small-VRAM training and quantified tradeoffs.
