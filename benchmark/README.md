# ComfyUI ROCm Benchmark Comparison

Little background from my [report](https://github.com/kyuz0/amd-strix-halo-comfyui-toolboxes/issues/24), that ComfyUI hogging CPU even when no task in the queue, resolved by using older PyTorch+ROCm 7.11. Why stop there let push with the latest ROCm 72.

This repository contains performance benchmarks for various ComfyUI Docker images utilizing AMD ROCm. Testing was conducted natively on a Geekom A9 Mega mini-PC running Fedora 43 Server, to evaluate workflow generation times across different image builds and ROCm driver versions.

## Benchmark Methodology

The benchmark tests were automated utilizing a dedicated workflow script. The complete execution logic, environment variables, and testing parameters are available in the [`benchmark_workflows.py`](https://github.com/muslimpribadi/amd-strix-halo-comfyui-toolboxes/blob/ec81a21f890f1a1044f5f49b8e9507a93012b1c4/scripts/benchmark_workflows.py).

## Docker Image Specifications

The testing suite evaluates one original base image and three modified forks to compare the impact of ROCm versions and container optimization strategies:

| Images | Build Size | Version | PyTorch URL |
| :--- | :--- | :--- | :--- |
| Original kyuz0 <br />[Dockerfile](https://github.com/muslimpribadi/amd-strix-halo-comfyui-toolboxes/blob/main/Dockerfile) | 15.1 GB| `PyTorch2.13.0a0 rocm7.14.0a20260529` | [`https://rocm.nightlies.amd.com/v2-staging/gfx1151`](https://rocm.nightlies.amd.com/v2-staging/gfx1151) |
| Local ROCm 72 <br />[Dockerfile.rocm72](https://github.com/muslimpribadi/amd-strix-halo-comfyui-toolboxes/blob/main/Dockerfile.rocm72) | 35.2 GB | `PyTorch2.13.0 rocm7.2` | [`https://download.pytorch.org/whl/rocm7.2`](https://download.pytorch.org/whl/rocm7.2) |
| Local ROCm 72 (multi-stage) <br />[Dockerfile.rocm72-slim](https://github.com/muslimpribadi/amd-strix-halo-comfyui-toolboxes/blob/main/Dockerfile.rocm72-slim) | 18 GB | `PyTorch2.13.0 rocm7.2` | [`https://download.pytorch.org/whl/rocm7.2`](https://download.pytorch.org/whl/rocm7.2) |
| Local ROCm 71 (multi-stage) <br />[Dockerfile.rocm71-slim](https://github.com/muslimpribadi/amd-strix-halo-comfyui-toolboxes/blob/main/Dockerfile.rocm71-slim) | 8 GB | `PyTorch2.11.0 rocm7.13.0` | [`https://repo.amd.com/rocm/whl/gfx1151/`](https://repo.amd.com/rocm/whl/gfx1151/) | 

> [!NOTE]
> *I also add pytorch+rocm 7.1 stable release for gfx1151, it's turnout as the smallest build.*
> <br />*All the fork version reach 🟢0% CPU load after task.*


## 📊 ComfyUI ROCm Builds Benchmark Comparison

This table compares the performance and size of four different ComfyUI ROCm podman images. The best results for each metric are marked with 🏆.

| Metric / Workflow | Original kyuz0<br>*(ROCm 7.14)* | Local ROCm 72<br>*(ROCm 7.2)* | Local ROCm 72 Slim<br>*(ROCm 7.2)* | Local ROCm 71 Slim<br>*(ROCm 7.13)* |
| :--- | :---: | :---: | :---: | :---: |
| `Flux1 Schnell 4 Steps` | **77 s** 🏆 | 78 s | 82 s | 82 s |
| `Hunyuan Video 1.5_720p_i2v 4 step lora` | **16 m 16 s** 🏆 | 16 m 48 s | 17 m 4 s | 18 m 6 s |
| `Hunyuan Video 1.5_720p_t2v 4 step lora` | **16 m 18 s** 🏆 | 16 m 27 s | 16 m 42 s | 17 m 16 s |
| `LTX2 I2V BF16` | 47 m 2 s | 18 m 4 s | **17 m 47 s** 🏆 | 19 m 34 s |
| `LTX2 T2V BF16` | 38 m 0 s | 20 m 36 s | **16 m 56 s** 🏆 | 17 m 31 s |
| `Qwen Image 2512 BF16 20 Steps` | **6 m 7 s** 🏆 | 7 m 8 s | 7 m 11 s | 6 m 41 s |
| `Qwen Image 2512 BF16 4 Step LoRA` | 10 m 51 s | 1 m 59 s | **1 m 53 s** 🏆 | 3 m 4 s |
| `Qwen Image Edit 2511 BF16 20 Steps` | **10 m 19 s** 🏆 | 12 m 1 s | 11 m 23 s | 10 m 38 s |
| `Qwen Image Edit 2511 BF16 4 Step LoRA` | 19 m 14 s | 3 m 12 s | **2 m 42 s** 🏆 | 3 m 28 s |
| `Wan2.2 I2V A14B 4steps lora rank64 Seko V1 FP16` | 31 m 39 s | **30 m 28 s** 🏆 | 31 m 22 s | 36 m 30 s |
| `Wan2.2 T2V A14B FP16 4steps lora rank64 Seko V2` | 29 m 42 s | **28 m 33 s** 🏆 | 29 m 3 s | 35 m 33 s |

### 💡 Interesting Facts & Observations

* **Size vs. Performance Sweet Spot**: The **Local ROCm 72 (Slim)** build effectively halves the image size (from 35.2 GB down to 18 GB) compared to the standard local build, without sacrificing generation speed—in fact, it even takes the crown for fastest LoRA performance. 
* **Tiny but Mighty**: The **Local ROCm 71 (Slim)** is remarkably small at just **8.18 GB** (an ~75% reduction from the standard build). While slightly slower on some heavy workflows like Wan2.2, it remains highly competitive across the board, making it perfect for storage-constrained setups.
* **Original Build's LTX2 & LoRA Bottlenecks**: The `Original kyuz0` image severely struggles with specific tasks, taking almost 47 minutes for `LTX2-I2V` and over 19 minutes for `Qwen-Image-Edit (LoRA)`. In contrast, the local builds breeze through these same tasks in roughly a third of the time (e.g., ~17 minutes and ~3 minutes, respectively).
* **Base Performance Baseline**: Surprisingly, for standard baseline generation without LoRAs (like `Flux1-Schnell` and `Qwen-Image-2512-BF16-20-Steps`), the heavier `Original kyuz0` staging build still performs the fastest, narrowly beating the custom local builds by a few seconds.

---

[ROCm and Linux Support on Strix Halo](https://www.youtube.com/watch?v=Hdg7zL3pcIs)
For additional context on running these AI workloads locally, you might find this discussion on configuring ROCm on Linux for Strix Halo hardware helpful in understanding the underlying driver and VGPR optimizations.
