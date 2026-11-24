<div align="center">

# Faster Kernel, Slower E2E (`faster-kernel-slower-e2e`)

**An Empirical Catalog & 9-Layer Taxonomy (`L0–L8`) of Operator-to-End-to-End Performance Gaps in AI Infrastructure**

[![Repos](https://img.shields.io/badge/Repositories-13-1d4ed8?style=flat-square)](#repository-catalog--分仓库台账导航)
[![Cases](https://img.shields.io/badge/Cases-152%20Perf%20%2F%20162%20Total-059669?style=flat-square)](#9-layer-root-cause-taxonomy--九层根因分布-l0l8)
[![Taxonomy](https://img.shields.io/badge/Taxonomy-L0--L8%20(9%20Layers)-7c3aed?style=flat-square)](#9-layer-root-cause-taxonomy--九层根因分布-l0l8)
[![Time Span](https://img.shields.io/badge/Time%20Span-2022--2026-475569?style=flat-square)](#key-empirical-findings--核心实证发现)

[Reports & Artifacts](#reports--artifacts--核心报告与汇总文档) • [Key Findings](#key-empirical-findings--核心实证发现) • [L0–L8 Taxonomy](#9-layer-root-cause-taxonomy--九层根因分布-l0l8) • [13-Repo Catalog](#repository-catalog--分仓库台账导航) • [Contributing](#contributing--贡献指南)

</div>

---

## Overview / 项目简介

In modern AI infrastructure (LLM serving engines, GPU/NPU operator libraries, deep learning compilers, and edge runtimes), an isolated kernel microbenchmark speedup frequently fails to translate into—or even **regresses**—end-to-end (E2E) training or serving performance.

This repository curates **162 real-world GitHub Issues, PRs, and engineering post-mortems (152 classified performance gap cases + 10 boundary/non-performance control records)** across **13 major open-source AI Infra repositories** (2022–2026), organized under a unified **9-layer root-cause taxonomy (`L0–L8`)** with quantitative isolated-vs-E2E metrics, profiling evidence, and community resolutions.

本项目持续整理现代 AI 基础设施中**「单算子微基准提速/持平，但端到端（E2E）模型训练或推理零收益、被稀释甚至性能倒退」**的真实工程案例。全库覆盖 **13 个主流开源仓库**（2022–2026 年），共归档 **162 条实证记录**（含 **152 例 `L0–L8` 有效性能案例** 与 10 例边界/非性能对照），每条记录均附原帖链接、软硬件环境、单算子与端到端双侧定量数据、Profiling 定位证据及社区最终解法。

---

## Reports & Artifacts / 核心报告与汇总文档

| Document | Format | Description |
| :--- | :---: | :--- |
| [**单算子到端到端性能鸿沟实证统计报告.md**](./reports/单算子到端到端性能鸿沟实证统计报告.md) | Markdown | 全库 152 例有效性能案例的完整多维统计报告（`L0–L8` 占比、13 仓分布矩阵、四大软件栈画像、复合耦合对、硬件平台、年度趋势、处置状态与九层防御清单） |
| [**单算子到端到端性能鸿沟实证统计报告.pdf**](./reports/单算子到端到端性能鸿沟实证统计报告.pdf) | PDF (4 pages) | 含 5 组高清矢量统计图（分布图、热力矩阵、软件栈堆叠图、耦合对与硬件图、年度与状态双联图）的排版报告 |

---

## Key Empirical Findings / 核心实证发现

- **65.1% of E2E gaps come from outside the kernel (`L1 + L3 + L4 + L5`)**: True intra-kernel microarchitectural regressions (`L6`, 11.2%) and cross-kernel hardware contention (`L7`, 5.3%) account for only **16.4%** of cases combined. The majority are caused by data-movement glue/layout/dequant (`L3`, **21.1%**), dispatch dead-paths/silent fallbacks (`L1`, **15.1%**), compiler/JIT/cache lifecycle overhead (`L4`, **14.5%**), and host launch floor (`L5`, **14.5%**).
  > **核外系统税主导性能鸿沟**：真正发生在核函数内部的微架构退化（`L6` 11.2%）与硬件资源争抢（`L7` 5.3%）合计仅占 **16.4%**；而 `L3` 胶水/布局、`L1` 路由死路径、`L4` 编译/JIT/缓存与 `L5` Host 发射四大外围系统层合计高达 **65.1%**（99/152 例）。
- **78.9% (120/152) are multi-factor coupled regressions**: On average each case involves **1.87 root-cause layers**. Top co-occurring pairs are **`L3 + L5`** (17 cases: layout/quant glue multiplying host launches), **`L4 + L5`** (14 cases: dynamic cache misses / JIT recompilation stalling host execution), and **`L1 + L6`** (11 cases: bad routing dispatching into mismatched microarchitectural kernels).
  > **高度多因耦合**：78.9% 的案例涉及两个及以上层级耦合；`L5`（2.32x）与 `L6`（2.53x）的复合放大倍数最高。
- **Hardware leaps amplify non-kernel taxes**: **68.4% (104/152)** of cases emerged in 2026 alongside NVIDIA Blackwell (33 mentions), Hopper (28), Huawei Ascend NPU (19), AMD ROCm (15), and Apple Silicon (11). Faster compute peaks shrink kernel time, exposing host launch, JIT, and padding overhead.
- **`L6` microarchitectural regressions trigger the most Reverts**: Out of **13 formal Reverts (8.6%)**, `L6` alone accounts for **5 (38.5%)**; **21.1% (32 cases)** remain `Open` awaiting cross-layer architectural fixes.

---

## 9-Layer Root-Cause Taxonomy / 九层根因分布 (`L0–L8`)

| Code | Category (EN / 中文) | Primary Cases | Share | Compound Mentions | Amplification | Typical Mechanisms |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- |
| **L0** | **Measurement & Proxy Distortion**<br>评测失真与代理偏差 | 9 | 5.9% | 13 | 1.44x | Hot-L2 microbench loops, uniform synthetic inputs, wall-clock wrapper mix, CUPTI observer overhead, key-only/single-slot blindspots |
| **L1** | **Dispatch Dead-Path & Silent Fallback**<br>路由死路径与静默回退 | 23 | 15.1% | 33 | 1.43x | Config blacklist (0% hit), unregistered Silent No-Op, swallowed ABI errors, silent SIMT/CPU fallback, batch-level worst-case veto |
| **L2** | **Critical-Path Dilution & Phase Asymmetry**<br>关键路径稀释与阶段掩盖 | 16 | 10.5% | 27 | 1.69x | Amdahl low-share dilution, masked off critical path by GEMM/comm, forward-vs-backward asymmetry, non-additive local speedups |
| **L3** | **Data-Movement Glue, Layout & Dequant**<br>胶水搬运、布局转换与反量化 | **32** | **21.1%** | **59** | 1.84x | Forced `.contiguous()` full-pool copies, Head/DP/Page zero-padding, NHWC↔NCHW reorder, unfused online quant/dequant, H2D copies |
| **L4** | **Compiler, JIT & Cache Lifecycle Tax**<br>图编译、JIT 重编与缓存生命周期反噬 | 22 | 14.5% | 34 | 1.55x | Graph breaks, Inductor FP64 widening, TVM over-inlining, per-step JIT recompile/GIL stalls, autotune timeouts, CUDA Graph VRAM tax |
| **L5** | **Host Dispatch & Small-Shape Launch Floor**<br>Host 发射底噪与小 Shape 启动下限 | 22 | 14.5% | 51 | 2.32x | Python/FFI/CustomOp per-call overhead, per-step plan recompute, sync bubbles, fragmented Peer-Copy storms, spec-decode step floor |
| **L6** | **Intra-Kernel Microarchitectural Degradation**<br>核内微架构退化 | 17 | 11.2% | 43 | **2.53x** | Register spill / occupancy cliffs, wave quantization & tail waves, cross-arch (CDNA/RDNA, SXM/PCIe, Ascend) tiling mismatch, ptxas scheduling |
| **L7** | **Cross-Kernel Contention, Cache & DVFS**<br>跨核资源争抢、缓存污染与功耗/频率反噬 | 8 | 5.3% | 17 | 2.12x | Shared matrix engine contention (M4 SME), concurrent stream SM/workspace contention, L1/L2 thrashing, power-cap / P-state frequency drop |
| **L8** | **Cross-Layer Numerical Amplification**<br>跨层数值误差放大与精度/接受率雪崩 | 3 | 2.0% | 7 | 2.33x | Deep-layer rounding error accumulation, TF32 truncation collapsing spec-decode acceptance, batch non-invariance, forced FP32 traffic doubling |
| **Total** | **Classified Performance Cases** | **152** | **100.0%** | **284** | **1.87x** | *+10 boundary / non-performance control records (162 total entries)* |

---

## Repository Catalog / 分仓库台账导航

Each `.xlsx` workbook contains **3 standardized sheets**: `案例总表` (15-column structured catalog with clickable GitHub Issue/PR hyperlinks), `统计看板` (live formula dashboard + charts), and `根因分类字典` (`L0–L8` taxonomy reference). `vllm_metal_kernel_vs_e2e_cases.xlsx` additionally includes a cross-repo sheet (`跨仓全景合并表`) merging `vLLM` + `vllm-metal`.

| Stack Layer | Repository | Workbook File | Total Entries | Perf Cases (`L0–L8`) | Dominant Root Causes |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **LLM Serving Engine**<br>大模型推理引擎 | **TensorRT-LLM** | [tensorrt_llm_kernel_vs_e2e_cases_master.xlsx](./cases/tensorrt_llm_kernel_vs_e2e_cases_master.xlsx) | 18 | 18 | `L3` (6), `L5` (4), `L2` (3) |
| | **vllm-ascend** | [vllm_ascend_kernel_vs_e2e_cases_master.xlsx](./cases/vllm_ascend_kernel_vs_e2e_cases_master.xlsx) | 17 | 17 | `L3` (5), `L5` (4), `L6` (3) |
| | **vLLM** | [vllm_kernel_vs_e2e_cases_master.xlsx](./cases/vllm_kernel_vs_e2e_cases_master.xlsx) | 16 | 16 | `L3` (4), `L1` (3), `L5` (3) |
| | **SGLang** | [sglang_kernel_vs_e2e_cases_master.xlsx](./cases/sglang_kernel_vs_e2e_cases_master.xlsx) | 15 | 15 | `L0` (3), `L1` (3), `L2/L3/L5/L6` (2) |
| **Operator Library**<br>算子库 | **FlashAttention** | [flash_attention_kernel_vs_e2e_cases_master.xlsx](./cases/flash_attention_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L1` (3), `L6` (3), `L4` (2) |
| | **FlashInfer** | [flashinfer_kernel_vs_e2e_cases_master.xlsx](./cases/flashinfer_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L4` (3), `L0` (2) |
| | **FlagGems** | [flaggems_kernel_vs_e2e_cases_master.xlsx](./cases/flaggems_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L1` (2), `L2` (2), `L5` (2) |
| | **Liger-Kernel** | [liger_kernel_vs_e2e_cases_master.xlsx](./cases/liger_kernel_vs_e2e_cases_master.xlsx) | 11 | 9 | `L4` (3), `L3` (2), `L5` (2) |
| **Framework & Compiler**<br>推理/编译框架 | **PyTorch** | [pytorch_kernel_vs_e2e_cases_master.xlsx](./cases/pytorch_kernel_vs_e2e_cases_master.xlsx) | 14 | 14 | `L4` (3), `L1/L3/L6` (2) |
| | **ONNX Runtime** | [onnxruntime_kernel_vs_e2e_cases_master.xlsx](./cases/onnxruntime_kernel_vs_e2e_cases_master.xlsx) | 10 | 10 | `L3` (6), `L1` (2), `L7` (2) |
| | **Apache TVM** | [tvm_operator_vs_e2e_cases_master.xlsx](./cases/tvm_operator_vs_e2e_cases_master.xlsx) | 13 | 6 | `L4` (4), `L1` (2) |
| **Local / Edge Engine**<br>端侧/本地引擎 | **llama.cpp** | [llama_cpp_kernel_vs_e2e_cases_master.xlsx](./cases/llama_cpp_kernel_vs_e2e_cases_master.xlsx) | 16 | 16 | `L6` (6), `L3` (3), `L1/L5/L7` (2) |
| | **vllm-metal** | [vllm_metal_kernel_vs_e2e_cases.xlsx](./cases/vllm_metal_kernel_vs_e2e_cases.xlsx) | 5 | 4 | `L2` (3), `L5` (1) |

---

## Schema Specification / 台账字段规范

All 13 workbooks share an identical 15-column schema (`A:O`) in `案例总表`:

1. `编号` (Case ID)
2. `Issue/PR` (Clickable GitHub URL)
3. `关联 PR / 追踪单` (Related PRs / Tracking Issues)
4. `状态归类` (`Closed` / `Mitigated` / `Open` / `Reverted` / `Tech Blog`)
5. `详细状态与版本` (Detailed Status & Release Version)
6. `年份` (Year) & `年月` (Year-Month)
7. `优化算子 / 特性` (Target Operator / Feature)
8. `硬件与 Workload` (Hardware & Model/Serving Workload)
9. `单算子 / 理论预期表现 (定量)` (Isolated Operator / Theoretical Metric)
10. `端到端 (E2E) 实测表现 (定量)` (Full-Model / Serving E2E Outcome)
11. `主根因` (`L0–L8`) & `复合根因` (Compound Root Causes)
12. `原帖 Profiling 定位证据` (Original Profiling & Root-Cause Evidence)
13. `最终处置与社区解法` (Final Disposition & Community Resolution)

---

## Contributing / 贡献指南

Pull Requests and Issues adding verified new cases are warmly welcome! To maintain empirical rigor, every new case should include:

1. **Direct Source Link**: A verifiable public GitHub Issue, PR, commit, or official engineering blog post.
2. **Dual-Side Evidence**: Both (a) isolated operator speedup or clear theoretical optimization expectation, and (b) full-model or serving-level E2E zero-gain, dilution, or regression with quantitative figures.
3. **Profiling Evidence & Classification**: Primary and compound root-cause labels under `L0–L8`, backed by profiler traces, counters, or source-level attribution from the original thread (no speculation).

欢迎通过 Issue 或 PR 补充更多真实案例！为保证实证质量，新增案例请提供**可追溯的公开链接**、**单算子预期与端到端实测双侧定量数据**以及**原帖 Profiling 定位依据**，并按 `L0–L8` 体系标注主根因与复合根因。
