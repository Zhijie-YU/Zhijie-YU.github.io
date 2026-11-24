# Cloth IPC Performance Benchmark

## 2026-11-24 · Optimized libuipc vs. libuipc-warp

### Configuration

- Newton `cloth_ipc` ran headless on Windows with an NVIDIA RTX PRO 5000 Blackwell Generation Laptop GPU. The scene used a 32-cell cloth grid, 1,089 cloth vertices, four boxes, and two simulation substeps per frame at `dt = 1/120 s`. The saved libuipc fixture matched a fresh Warp scene export array for array.
- Each configuration ran three 120-frame trials in rotated order in the same benchmark batch. The first 10 frames of each trial were excluded, leaving 330 frame times per configuration. The timer starts before the two substeps and stops after GPU synchronization, using `world.sync()` for libuipc and `wp.synchronize_device()` for Warp. Rendering and trajectory export run outside the timed benchmark.
- Warp Graph-on enabled the solver's substep Graph. Warp Graph-off disabled both capture layers. To bring the corresponding assembly stages closer to libuipc-warp, the local libuipc CUDA build uses low-rank friction Hessian assembly and upper-triangular block coordinate (BCOO) matrix conversion. The optimized Release DLL ran with `linear_system/use_cuda_graph=0`. Warp used Newton `56ff670`, libuipc-warp `f70954b`, and Warp `1.20.0.dev20261117`.

### Results

| Configuration | Trial means (ms/frame) | Mean (ms/frame) | Median (ms/frame) | P95 (ms/frame) |
| --- | --- | ---: | ---: | ---: |
| libuipc-warp, substep Graph enabled | 7.25 / 6.58 / 6.32 | 6.72 | 4.75 | 14.30 |
| libuipc-warp, CUDA Graph disabled | 71.85 / 64.70 / 63.54 | 66.70 | 48.71 | 156.18 |
| Optimized libuipc, CUDA Graph disabled (mode 0) | 90.96 / 93.85 / 92.85 | 92.55 | 63.86 | 198.99 |

### Conclusions

- Enabling Warp's substep Graph reduced its mean frame time by a factor of **9.93**.
- With Graph disabled in both backends, optimized libuipc took **1.39x** as long as Warp. Optimized libuipc took **13.78x** as long as Warp with its substep Graph.

[Raw results and build hash](C:/Work/Newton-IPC/.cache/cloth-synced-ab/results.json) · [Timing harness](C:/Work/Newton-IPC/.cache/cloth-synced-ab/benchmark.py)
