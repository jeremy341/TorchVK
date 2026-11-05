# TorchVK media-heavy validation fixture

A small README validation fixture.

![GPU](https://example.com/gpu.png)
![Training chart](https://example.com/chart.png)
![Dashboard](https://example.com/dashboard.png)
![Results](https://example.com/results.png)
![Logo](https://example.com/logo.png)
![Screenshot](https://example.com/screenshot.png)

[![Build](https://img.shields.io/badge/build-test-blue)](https://example.com/build)
[![License](https://img.shields.io/badge/license-test-green)](https://example.com/license)

```python
import torch

for step in range(10):
    loss = model(data).square().mean()
    loss.backward()
    optimizer.step()
```