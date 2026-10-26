# TorchVK hardware notes

This file explains why older AMD consumer GPUs are the target hardware. The build plan is in [PROJECT.md](PROJECT.md), and the research framing is in [RESEARCH.md](RESEARCH.md).

## The argument

Two different things get called "better for AI," and they should be kept apart:

- Raw FP32 throughput, memory bandwidth and VRAM. These matter directly to a custom Vulkan backend.
- Existing PyTorch performance. NVIDIA usually wins here because CUDA, cuDNN and Tensor Cores are heavily optimized.

The working hypothesis is that many AMD GPUs have useful compute that goes unused because the software stack for them is weak. If that is right, the bottleneck is software availability, not hardware capability.

## Development GPU: RX 5600 XT

Launch price was $279.

| Spec | Value |
|---|---|
| Stream processors | 2304 |
| FP32 | 7.19 TFLOPS (manufacturer figure) |
| FP16 | 14.38 TFLOPS (manufacturer figure) |
| VRAM | 6 GB GDDR6, 192-bit |
| Bandwidth | up to 336 GB/s |
| TDP | 150 W |

### Reference competitor: RTX 2060

Launch price was $349. Specs: 1920 CUDA cores, about 6.5 FP32 TFLOPS, 6 GB GDDR6, 336 GB/s, 160 W, and 240 Tensor Cores.

On paper the RX 5600 XT has about 11% more FP32 throughput for about 20% less money, with the same memory bandwidth. The RTX 2060 is still the card most people could train models on, because of its software support. This makes the RX 5600 XT a good first test: if TorchVK can use even part of its compute, the gap is software, not hardware.

## Example: Vega 56 and GTX 1070

Vega 56 ($399): 10.5 FP32 TFLOPS, 8 GB HBM2, 410 GB/s.
GTX 1070 ($379): about 6.75 FP32 TFLOPS, 8 GB GDDR5, 256 GB/s.

The Vega card had roughly 56% more theoretical compute and 60% more bandwidth for about the same price. In 2018 the usual advice was still to buy the GTX 1070. The reason was the software stack, not the hardware:

```text
GTX 1070 -> CUDA -> cuDNN -> PyTorch -> optimized kernels -> fast
Vega 56  -> capable hardware -> little ML software support -> hard going
```

The project asks how much of that unused compute Vulkan can expose for ML training.

## AMD generations

- **Polaris and Vega** (RX 470/480/570/580, Vega 56/64): large FP32 and, for Vega, HBM2 bandwidth, compared with similarly priced Pascal cards.
- **RDNA1** (RX 5500 XT, 5600 XT, 5700, 5700 XT): good compute per dollar. The RX 5700 XT was $399 against about $499 for an RTX 2070 Super-class card.
- **RDNA2 and later**: NVIDIA's Ampere FP32 gains mean Radeon no longer leads on raw shader throughput. For example, the RX 6600 XT is listed at 10.6 TF and the RTX 3060 at about 13 TF. AMD often shipped more VRAM per tier, though. The RX 6800 has 16 GB where the RTX 3070 has 8 GB. A model that fits in VRAM runs, and one that does not, does not.

RDNA3 example: the RX 7900 XTX lists 61.4 FP32 TFLOPS with 24 GB at $999. The RTX 4080 lists about 49 shader TFLOPS with 16 GB at $1,199. On paper AMD wins on every number, but the RTX 4080 often still wins on ML work because of Tensor Cores and CUDA libraries.

## Caveat: FP32 is not the whole story

Newer NVIDIA cards pair modest shader throughput with dedicated matrix units. The RTX 4060, for example, lists about 15 shader TFLOPS and about 242 AI TOPS from Tensor Cores. A simple Vulkan kernel uses ordinary shader ALUs, so raw FP32 comparisons overstate AMD's position against current NVIDIA hardware.

This does not weaken the project. It sets the scope: TorchVK targets older cards that have no dedicated ML path at all. AMD's RDNA4 adds matrix hardware on its side, such as the RX 9060 XT (103 FP16 and 205 FP8 matrix TFLOPS) and the RX 9070 XT (195 FP16 and 389 FP8). A mature version might eventually use these through Vulkan.

## Candidate future targets

| GPU | Reason to consider it |
|---|---|
| RX 580 8 GB | Cheap, 8 GB, strong FP32 |
| Vega 56 | 10.5 TF and 410 GB/s for $399 |
| Vega 64 | About 12.7 TF and 484 GB/s |
| RX 5600 XT | Development target: 7.19 TF at $279, against 6.5 TF at $349 for the RTX 2060 |
| RX 5700 XT | Strong FP32 per dollar |
| RX 6700 XT | 12 GB VRAM |
| RX 6800 / 6800 XT | 16 GB VRAM |
| RX 7800 XT | 16 GB at $499 |
| RX 7900 XTX | 24 GB and 61.4 TFLOPS |
| RX 9060 XT 16 GB | Inexpensive 16 GB with matrix hardware |
| Intel Arc | Cross-vendor check |

Testing other vendors and multiple GPUs is a long-term goal. It is not needed for v1, which targets Windows and the RX 5600 XT only.

## CPU baseline

The first comparison is an AMD Ryzen 5 5600 against an RX 5600 XT running TorchVK.

## Measurement

The spec numbers above are manufacturer figures. They motivate the project and are not results. Each kernel stage will report achieved throughput as a percentage of the theoretical peak:

```text
Naive GEMM      x.x TFLOPS    x% of 7.19 TF
Tiled GEMM      x.x TFLOPS    x%
Optimized       x.x TFLOPS    x%
```

Values are filled in only after they have been measured.
