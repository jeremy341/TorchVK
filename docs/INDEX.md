# TorchVK documentation

This directory is also an Obsidian vault. All Markdown is tracked in Git. Personal Obsidian settings are not.

- [PROJECT_CHARTER.md](PROJECT_CHARTER.md): goal, scope, contribution and feasibility criteria
- [RELATED_WORK.md](RELATED_WORK.md): existing alternatives and the risk that the work is not new
- [ARCHITECTURE.md](ARCHITECTURE.md): candidate PyTorch and Vulkan layers, and the alternatives
- [ROADMAP.md](ROADMAP.md): gated milestones with rough time estimates
- [EVALUATION.md](EVALUATION.md): benchmark protocol and claims that can be made
- [DECISIONS.md](DECISIONS.md): open questions and dated decisions
- [LEARNING.md](LEARNING.md): study resources and papers
- [EXPERIMENT_TEMPLATE.md](EXPERIMENT_TEMPLATE.md): template for experiment notes
- [experiments/](experiments/): dated experiment records, starting with the R0 audit

Status as of 2026-10-10: research and planning. The original repository held only a README heading and a license. Check the current state of the repository before starting a task.

## Glossary

- **Backend:** a platform-specific implementation of tensor operations.
- **PrivateUse1:** the PyTorch extension point for custom accelerator backends.
- **GEMM:** general matrix multiplication.
- **SPIR-V:** the binary intermediate format for GPU compute shaders.
- **Autograd:** PyTorch's automatic gradient computation.
- **CPU fallback:** an operation the device does not support runs on the CPU instead. This can hide missing GPU support.
- **Ablation:** a controlled test that changes or removes one optimization.

## Research stance

The question is not whether PyTorch can train through Vulkan. Existing packages already show that. The question is what can be improved for older AMD hardware under Windows, with strict verification, small VRAM budgets, and measured trade-offs.
