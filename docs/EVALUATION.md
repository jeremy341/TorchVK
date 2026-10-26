# Benchmarking and reproducibility protocol

## Baselines

1. Optimized PyTorch on the CPU (Ryzen 5 5600).
2. PyTorch DirectML on the RX 5600 XT, where the chosen model and operators are supported.
3. The existing [torchvulkan](https://github.com/zainsharief/torchvulkan) on the RX 5600 XT, with `TORCHVULKAN_STRICT=1` where it is supported.
4. TorchVK, or an upstream optimization, but only once it works.

## Controls

For backend comparisons, keep the model, weights and initialization, dataset, preprocessing, loss, batch size, optimizer, step count and precision identical. A separate mixed-precision comparison can be run for practical reasons. Do not mix it up with the same-precision benchmark.

The published torchvulkan benchmark uses FP16 on the GPU and FP32 on the CPU. That is useful for an end-user comparison, but it does not isolate software speed at equal precision. Include an FP32-matched analysis.

Use warmup runs, synchronize before reading timers, take several repeats, and record the system and its load. Report GPU kernel time, transfer time and end-to-end wall-clock time separately.

Do not claim GPU-only training unless operator tracing and strict fallback checks confirm that forward, backward, loss and the optimizer all stay on the device.

## Metrics

- Correctness: maximum absolute and relative error, gradient agreement, convergence curves and held-out accuracy.
- Performance: latency, samples per second, dispatch time, transfer time, GPU kernel time and throughput.
- Memory: peak VRAM, allocator overhead, host RAM, and the largest batch or model that fits.
- Completeness: native operations, unsupported operations, CPU fallbacks, and GPU-CPU copy events.
- Environment: OS, GPU and PCI ID, driver, Vulkan SDK or runtime, compiler, Python and PyTorch versions, exact git commit, and shader flags.
- Energy: power draw, joules per step, and joules to reach a target accuracy. Report these only if the instrumentation is reliable.

## Experiment matrix

- E1: vector add over 1K to 10M FP32 elements, including odd sizes.
- E2: GEMM at shapes from 128 to 4096, including rectangular cases. Compare naive and tiled kernels at matched precision.
- E3: MNIST MLP forward, backward and optimizer steps with strict GPU execution.
- E4: a modest CIFAR-10 CNN, forward and backward, with fair baselines.
- E5: small-VRAM sensitivity, covering batch size and workspace use.
- E6: ablations that change one optimization at a time: tiling, dispatch batching, caching, memory strategy.

## Raw output fields

```text
experiment_id,utc_timestamp,repo,git_sha,backend,gpu_name,driver_version,
vulkan_version,os,torch_version,dtype,model,dataset,shape,batch_size,
warmup,repeats,seed,strict_gpu_only,cpu_fallback_ops,kernel_ms,
transfer_ms,total_step_ms,vram_peak_mb,loss,accuracy,error,status
```

Publish failures and excluded runs. A result without its configuration and provenance is not a research result. Report medians and spread, not only the fastest run.
