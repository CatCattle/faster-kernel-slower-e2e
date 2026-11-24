<div align="center">

# Faster Kernel, Slower E2E (`faster-kernel-slower-e2e`)

**An Empirical Catalog & 9-Layer Taxonomy (`L0–L8`) of Operator-to-End-to-End Performance Gaps in AI Infrastructure**

[![Repos](https://img.shields.io/badge/Repositories-13-1d4ed8?style=flat-square)](#repository-catalog)
[![Cases](https://img.shields.io/badge/Cases-152%20Perf%20%2F%20162%20Total-059669?style=flat-square)](#9-layer-root-cause-taxonomy-l0l8)
[![Taxonomy](https://img.shields.io/badge/Taxonomy-L0--L8%20(9%20Layers)-7c3aed?style=flat-square)](#9-layer-root-cause-taxonomy-l0l8)
[![Time Span](https://img.shields.io/badge/Time%20Span-2022--2026-475569?style=flat-square)](#key-empirical-findings)

**English** | [简体中文](./README_ZH.md)

[Reports & Artifacts](#reports--artifacts) • [Key Findings](#key-empirical-findings) • [L0–L8 Taxonomy](#9-layer-root-cause-taxonomy-l0l8) • [13-Repo Catalog](#repository-catalog) • [Schema](#schema-specification) • [Contributing](#contributing)

</div>

---

## Overview

In modern AI infrastructure—spanning LLM serving engines, GPU/NPU operator libraries, deep learning compilers, and edge runtimes—an isolated kernel microbenchmark speedup frequently fails to translate into, or even **regresses**, end-to-end (E2E) training or serving performance.

This repository curates **162 real-world GitHub Issues, PRs, and official engineering post-mortems (152 classified performance gap cases + 10 boundary/non-performance control records)** across **13 major open-source AI Infra repositories** (2022–2026). Every entry is structured under a unified **9-layer root-cause taxonomy (`L0–L8`)** with clickable source links, hardware/workload context, quantitative isolated-vs-E2E metrics, profiling evidence, and community resolutions.

---

## Reports & Artifacts

| Document | Format | Description |
| :--- | :---: | :--- |
| [**Empirical Statistical Report (Markdown)**](./reports/单算子到端到端性能鸿沟实证统计报告.md) | Markdown | Full multi-dimensional analysis of all 152 classified cases (`L0–L8` shares, 13-repo matrix, 4 stack-layer profiles, co-occurrence pairs, hardware distribution, 2022–2026 trend, resolution status, and 9-layer defense checklist) |
| [**Empirical Statistical Report (PDF)**](./reports/单算子到端到端性能鸿沟实证统计报告.pdf) | PDF (4 pages) | Publication-ready 4-page A4 report featuring 5 vector charts (distribution, repo×taxonomy heatmap, stack-layer stacked bars, coupling pairs & hardware, temporal trend & resolution status) |

---

## Key Empirical Findings

- **65.1% of E2E performance gaps originate outside the kernel (`L1 + L3 + L4 + L5`)**: True intra-kernel microarchitectural regressions (`L6`, 11.2%) and cross-kernel hardware contention (`L7`, 5.3%) account for only **16.4%** of cases combined. The majority stem from data-movement glue, layout transforms, and dequantization (`L3`, **21.1%**), dispatch dead-paths and silent fallbacks (`L1`, **15.1%**), compiler/JIT/cache lifecycle overhead (`L4`, **14.5%**), and host launch floors (`L5`, **14.5%**).
- **78.9% (120/152) are multi-factor coupled regressions**: On average, each case involves **1.87 root-cause layers**. The most frequent co-occurring pairs are **`L3 + L5`** (17 cases: layout/quantization glue multiplying host launches), **`L4 + L5`** (14 cases: dynamic cache misses or JIT recompilation stalling host execution), and **`L1 + L6`** (11 cases: bad routing dispatching into mismatched microarchitectural kernels).
- **Hardware leaps amplify non-kernel taxes**: **68.4% (104/152)** of cases emerged in 2026 alongside NVIDIA Blackwell (33 mentions), Hopper (28), Huawei Ascend NPU (19), AMD ROCm (15), and Apple Silicon (11). As compute peaks rise and kernel durations shrink to microseconds, host launch, JIT, and padding overhead increasingly dominate E2E latency.
- **`L6` microarchitectural regressions trigger the most Reverts**: Out of **13 formal Reverts (8.6%)**, `L6` alone accounts for **5 (38.5%)**; meanwhile, **21.1% (32 cases)** remain `Open` awaiting cross-layer architectural fixes.

---

## 9-Layer Root-Cause Taxonomy (`L0–L8`)

| Code | Category | Primary Cases | Share | Compound Mentions | Amplification | Typical Mechanisms |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- |
| **L0** | **Measurement & Proxy Distortion** | 9 | 5.9% | 13 | 1.44x | Hot-L2 microbench loops, uniform synthetic inputs, wall-clock wrapper mixing, CUPTI observer overhead, key-only/single-slot blindspots |
| **L1** | **Dispatch Dead-Path & Silent Fallback** | 23 | 15.1% | 33 | 1.43x | Config blacklist (0% hit), unregistered Silent No-Op, swallowed ABI errors, silent SIMT/CPU fallback, batch-level worst-case veto |
| **L2** | **Critical-Path Dilution & Phase Asymmetry** | 16 | 10.5% | 27 | 1.69x | Amdahl low-share dilution, masked off critical path by GEMM/comm, forward-vs-backward asymmetry, non-additive local speedups |
| **L3** | **Data-Movement Glue, Layout & Dequant** | **32** | **21.1%** | **59** | 1.84x | Forced `.contiguous()` full-pool copies, Head/DP/Page zero-padding, NHWC↔NCHW reorder, unfused online quant/dequant, H2D copies |
| **L4** | **Compiler, JIT & Cache Lifecycle Tax** | 22 | 14.5% | 34 | 1.55x | Graph breaks, Inductor FP64 widening, TVM over-inlining, per-step JIT recompile/GIL stalls, autotune timeouts, CUDA Graph VRAM tax |
| **L5** | **Host Dispatch & Small-Shape Launch Floor** | 22 | 14.5% | 51 | 2.32x | Python/FFI/CustomOp per-call overhead, per-step plan recompute, sync bubbles, fragmented Peer-Copy storms, spec-decode step floor |
| **L6** | **Intra-Kernel Microarchitectural Degradation** | 17 | 11.2% | 43 | **2.53x** | Register spill / occupancy cliffs, wave quantization & tail waves, cross-arch (CDNA/RDNA, SXM/PCIe, Ascend) tiling mismatch, ptxas scheduling |
| **L7** | **Cross-Kernel Contention, Cache & DVFS** | 8 | 5.3% | 17 | 2.12x | Shared matrix engine contention (M4 SME), concurrent stream SM/workspace contention, L1/L2 thrashing, power-cap / P-state frequency drop |
| **L8** | **Cross-Layer Numerical Amplification** | 3 | 2.0% | 7 | 2.33x | Deep-layer rounding error accumulation, TF32 truncation collapsing spec-decode acceptance, batch non-invariance, forced FP32 traffic doubling |
| **Total** | **Classified Performance Cases** | **152** | **100.0%** | **284** | **1.87x** | *+10 boundary / non-performance control records (162 total entries)* |

---

## Repository Catalog

Each `.xlsx` workbook in [`cases/`](./cases/) contains **3 standardized sheets**: `案例总表` (15-column structured catalog with clickable GitHub Issue/PR hyperlinks), `统计看板` (live formula dashboard + charts), and `根因分类字典` (`L0–L8` taxonomy reference). `vllm_metal_kernel_vs_e2e_cases.xlsx` also includes a cross-repo sheet (`跨仓全景合并表`) merging `vLLM` and `vllm-metal`.

| Stack Layer | Repository | Workbook File | Total Entries | Perf Cases (`L0–L8`) | Dominant Root Causes |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **LLM Serving Engine** | **TensorRT-LLM** | [tensorrt_llm_kernel_vs_e2e_cases_master.xlsx](./cases/tensorrt_llm_kernel_vs_e2e_cases_master.xlsx) | 18 | 18 | `L3` (6), `L5` (4), `L2` (3) |
| | **vllm-ascend** | [vllm_ascend_kernel_vs_e2e_cases_master.xlsx](./cases/vllm_ascend_kernel_vs_e2e_cases_master.xlsx) | 17 | 17 | `L3` (5), `L5` (4), `L6` (3) |
| | **vLLM** | [vllm_kernel_vs_e2e_cases_master.xlsx](./cases/vllm_kernel_vs_e2e_cases_master.xlsx) | 16 | 16 | `L3` (4), `L1` (3), `L5` (3) |
| | **SGLang** | [sglang_kernel_vs_e2e_cases_master.xlsx](./cases/sglang_kernel_vs_e2e_cases_master.xlsx) | 15 | 15 | `L0` (3), `L1` (3), `L2/L3/L5/L6` (2) |
| **Operator Library** | **FlashAttention** | [flash_attention_kernel_vs_e2e_cases_master.xlsx](./cases/flash_attention_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L1` (3), `L6` (3), `L4` (2) |
| | **FlashInfer** | [flashinfer_kernel_vs_e2e_cases_master.xlsx](./cases/flashinfer_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L4` (3), `L0` (2) |
| | **FlagGems** | [flaggems_kernel_vs_e2e_cases_master.xlsx](./cases/flaggems_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L1` (2), `L2` (2), `L5` (2) |
| | **Liger-Kernel** | [liger_kernel_vs_e2e_cases_master.xlsx](./cases/liger_kernel_vs_e2e_cases_master.xlsx) | 11 | 9 | `L4` (3), `L3` (2), `L5` (2) |
| **Framework & Compiler** | **PyTorch** | [pytorch_kernel_vs_e2e_cases_master.xlsx](./cases/pytorch_kernel_vs_e2e_cases_master.xlsx) | 14 | 14 | `L4` (3), `L1/L3/L6` (2) |
| | **ONNX Runtime** | [onnxruntime_kernel_vs_e2e_cases_master.xlsx](./cases/onnxruntime_kernel_vs_e2e_cases_master.xlsx) | 10 | 10 | `L3` (6), `L1` (2), `L7` (2) |
| | **Apache TVM** | [tvm_operator_vs_e2e_cases_master.xlsx](./cases/tvm_operator_vs_e2e_cases_master.xlsx) | 13 | 6 | `L4` (4), `L1` (2) |
| **Local / Edge Engine** | **llama.cpp** | [llama_cpp_kernel_vs_e2e_cases_master.xlsx](./cases/llama_cpp_kernel_vs_e2e_cases_master.xlsx) | 16 | 16 | `L6` (6), `L3` (3), `L1/L5/L7` (2) |
| | **vllm-metal** | [vllm_metal_kernel_vs_e2e_cases.xlsx](./cases/vllm_metal_kernel_vs_e2e_cases.xlsx) | 5 | 4 | `L2` (3), `L5` (1) |

---

## Schema Specification

All 13 workbooks follow an identical 15-column schema (`A:O`):

1. **Case ID** (`编号`)
2. **Issue / PR** (`Issue/PR`, clickable GitHub URL)
3. **Related PR / Tracking Issue** (`关联 PR / 追踪单`)
4. **Status Category** (`状态归类`: `Closed` / `Mitigated` / `Open` / `Reverted` / `Tech Blog`)
5. **Detailed Status & Version** (`详细状态与版本`)
6. **Year & Year-Month** (`年份`, `年月`)
7. **Target Operator / Feature** (`优化算子 / 特性`)
8. **Hardware & Workload** (`硬件与 Workload`)
9. **Isolated Operator / Theoretical Metric** (`单算子 / 理论预期表现 (定量)`)
10. **End-to-End (E2E) Outcome** (`端到端 (E2E) 实测表现 (定量)`)
11. **Primary & Compound Root Cause** (`主根因`, `复合根因` under `L0–L8`)
12. **Original Profiling Evidence** (`原帖 Profiling 定位证据`)
13. **Final Disposition & Community Resolution** (`最终处置与社区解法`)

---

## Contributing

Pull Requests and Issues adding verified new cases are warmly welcome! To maintain empirical rigor, every submission should include:

1. **Direct Source Link**: A verifiable public GitHub Issue, PR, commit, or official engineering blog post.
2. **Dual-Side Evidence**: Both (a) isolated operator speedup or clear theoretical optimization expectation, and (b) full-model or serving-level E2E zero-gain, dilution, or regression with quantitative figures.
3. **Profiling Evidence & Taxonomy**: Primary and compound root-cause labels under `L0–L8`, backed by profiler traces, hardware counters, or source-level attribution from the original thread (no speculation).
