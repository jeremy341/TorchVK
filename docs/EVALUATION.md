# Benchmarking and reproducible research protocol

## Baselines
1. Optimized PyTorch CPU on Ryzen 5 5600.
2. PyTorch DirectML on RX 5600 XT if the selected model/ops are supported.
3. Existing [torchvulkan](https://github.com/zainsharief/torchvulkan) on the same RX 5600 XT with `TORCHVULKAN_STRICT=1` where supported.
4. TorchVK or an upstream optimization only **when functional**.

## Scientific controls
Same model, weights/initialization, dataset, preprocessing, loss, batch, optimizer, steps and precision for isolated backend comparisons. Run a separate practical mixed-precision comparison if desired; don't confuse it with identical-precision benchmarking.

The published torchvulkan benchmark code uses **FP16 for GPU and FP32 for CPU**. That is useful for a practical end-user contrast but it does not isolate software-backend speed at equal precision. Include an FP32-matched analysis.

Use warmups, synchronize when timing, multiple repeats, and record system/load context. Separate GPU-resident kernel timing, transfer overhead, and end-to-end wall clock.

No claim of GPU-only training unless operator tracing and strict fallback checks confirm forward, backward, loss and optimizer remain on device.

## Metrics
- Correctness: maximum absolute and relative numerical error, gradient consistency, convergence curves and held-out accuracy.
- Performance: latency, samples/sec, dispatch time, transfers, GPU kernel time, throughput.
- Memory: peak VRAM, allocator overhead, host RAM, largest workable batch/model.
- Completeness: native operations, failed/unsupported ops, CPU fallbacks, GPU/CPU copy events.
- Environment: OS, GPU & PCI ID, driver, Vulkan SDK/runtime, compiler, Python/PyTorch versions, exact git SHA and shader flags.
- Energy: power, joules/step and joules-to-target-accuracy **only if reliable instrumentation exists**.

## Experiment matrix
E1 vector-add across 1K–10M FP32 elements, including odd sizes.
E2 GEMM shapes 128–4096 and rectangular cases; naive vs tiled; matched precision.
E3 MNIST MLP forward/backward/optimizer with strict GPU execution.
E4 modest CIFAR-10 CNN forward/backward with fair baselines.
E5 small-VRAM sensitivity, batch size and workspace usage.
E6 ablation: change **one optimization** at a time (tiling, dispatch batching, caching, memory strategy).

## Raw output fields
```text
experiment_id,utc_timestamp,repo,git_sha,backend,gpu_name,driver_version,
vulkan_version,os,torch_version,dtype,model,dataset,shape,batch_size,
warmup,repeats,seed,strict_gpu_only,cpu_fallback_ops,kernel_ms,
transfer_ms,total_step_ms,vram_peak_mb,loss,accuracy,error,status
```

Publish failures and exclusions. A result without configuration and provenance is not a research result. Report medians and variability rather than a single fastest run.
