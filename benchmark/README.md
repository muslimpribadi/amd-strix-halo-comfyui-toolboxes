# ComfyUI ROCm Benchmark Comparison

This repository contains performance benchmarks for various ComfyUI Docker images utilizing AMD ROCm. Testing was conducted natively on a Geekom A9 Mega mini-PC running Fedora 43 Server, to evaluate workflow generation times across different image builds and ROCm driver versions.

## Benchmark Methodology

The benchmark tests were automated utilizing a dedicated workflow script. The complete execution logic, environment variables, and testing parameters are available in the [`benchmark_workflows.py`](https://github.com/muslimpribadi/amd-strix-halo-comfyui-toolboxes/blob/ec81a21f890f1a1044f5f49b8e9507a93012b1c4/scripts/benchmark_workflows.py).

## Docker Image Specifications

The testing suite evaluates one original base image and two modified forks to compare the impact of ROCm versions and container optimization strategies:

* **RoCm71 (Original)**: The original base image (`docker.io/kyuz0/amd-strix-halo-comfyui:latest`) specifically optimized for Strix Halo architectures. It pulls from ROCm Nightly `v2-staging/gfx1151` (tracking versions ~7.10 - 7.13). The total image size is 💾 15.1 GB.
* **RoCm72 (Fork)**: A forked image (`localhost/comfyui-rocm:rocm72`) utilizing the stable PyTorch ROCm 7.2 release (`whl/rocm7.2`). The total image size is 💾 35.2 GB.
* **RoCm72-Slim (Fork)**: A refined, multi-stage build of the ROCm 7.2 fork (`localhost/comfyui-rocm:rocm72-slim`) designed to significantly reduce the storage footprint. The total image size is 💾 18.0 GB.

## Benchmark Results

All durations below represent the total execution time in seconds (`duration_seconds`) for each workflow, rounded to two decimal places.

| Workflow | <a id="ref1">RoCm71</a> <br />`[1]` | <a id="ref1">RoCm72</a> <br />`[2]` | <a id="ref3">RoCm72-Slim</a> <br />`[3]` |
| --- | --- | --- | --- |
| Flux1-Schnell-4-Steps | 77.77 | 78.53 | 82.40 |
| Hunyuan-Video-1.5_720p_i2v-4-step | 976.25 | 1008.85 | 1024.50 |
| Hunyuan-Video-1.5_720p_t2v-4-step | 978.46 | 987.46 | 1002.14 |
| LTX2-I2V-BF16 | 2822.08 | 1084.53 | 1067.46 |
| LTX2-T2V-BF16 | 2280.19 | 1236.49 | 1016.79 |
| Qwen-Image-2512-BF16-20-Steps | 367.45 | 428.32 | 431.74 |
| Qwen-Image-2512-BF16-4-Step-LoRA | 651.79 | 119.34 | 113.47 |
| Qwen-Image-Edit-2511-BF16-20-Steps | 619.61 | 721.22 | 683.50 |
| Qwen-Image-Edit-2511-BF16-4-Step-LoRA | 1154.68 | 192.51 | 162.58 |
| Wan2.2-I2V-A14B-4steps-lora | 1899.99 | 1828.69 | 1882.42 |
| Wan2.2-T2V-A14B-FP16-4steps-lora | 1782.70 | 1713.66 | 1743.08 |

## Insights and Observations

*   **The LTX2 and LoRA Performance Bottleneck:** <br />The Strix Halo (`RoCm71`) image experiences a massive performance penalty on specific workflows. <br />It requires 2822.08 seconds to process the LTX2-I2V-BF16 workflow[[1]](#ref1), which is over 2.5x slower than the ROCm 7.2 build at 1084.53 seconds[[2]](#ref2). <br />It also suffers a drastic slowdown during Qwen-Image-Edit 4-Step LoRA runs, taking 1154.68 seconds[[1]](#ref1) compared to just 192.51 seconds on the standard ROCm 7.2 build[[2]](#ref2). <br />This points to a likely optimization gap or fallback issue within the nightly driver for specific attention mechanisms or PEFT operations.
*   **Strong Baseline Model Performance:** <br />When avoiding LoRAs or LTX2 workflows, the Strix Halo (`RoCm71`) nightly build matches or beats the stable ROCm 7.2 builds[[1]](#ref1)[[2]](#ref2). <br />For example, the base Qwen-Image 20-Steps completes faster on Strix Halo  (`RoCm71`) (367.45 seconds)[[1]](#ref1) than on ROCm 7.2 (428.32 seconds)[[2]](#ref2). <br />Flux1-Schnell is also slightly faster on Strix Halo (`RoCm71`) at 77.77 seconds[[1]](#ref1) versus 78.53 seconds on ROCm 7.2[[2]](#ref2).
*   **The "Slim" Image Efficiency:** <br />The `RoCm72-Slim` image maintains top-tier performance despite drastically reducing the container footprint. <br />It actually outperforms the base `RoCm72` image in several scenarios, including LTX2-T2V where it finishes in 1016.79 seconds[[3]](#ref3) compared to 1236.49 seconds[[2]](#ref2). <br />It also yields faster times for the Qwen-Image-Edit 4-Step LoRA at 162.58 seconds[[3]](#ref3) compared to the full ROCm 7.2 image at 192.51 seconds[[2]](#ref2).
*   **Hunyuan Video Consistency:** <br />Execution times for Hunyuan-Video are remarkably stable across all three tested environments. <br />The 720p text-to-video workflow times remain incredibly close across the board, scaling from 978.46 seconds on Strix Halo (`RoCm71`)[[1]](#ref1), to 987.46 seconds on ROCm 7.2[[2]](#ref2), and 1002.14 seconds on the Slim image[[3]](#ref3).

---

[ROCm and Linux Support on Strix Halo](https://www.youtube.com/watch?v=Hdg7zL3pcIs)
For additional context on running these AI workloads locally, you might find this discussion on configuring ROCm on Linux for Strix Halo hardware helpful in understanding the underlying driver and VGPR optimizations.
