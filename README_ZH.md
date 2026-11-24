<div align="center">

# 单算子加速到端到端（E2E）性能鸿沟实证案例库（`faster-kernel-slower-e2e`）

**覆盖 13 个主流开源 AI Infra 仓库的 152+ 真实工程案例与九层根因分类体系（`L0–L8`）**

[![仓库数量](https://img.shields.io/badge/覆盖仓库-13-1d4ed8?style=flat-square)](#分仓库台账导航)
[![案例规模](https://img.shields.io/badge/实证案例-152%20有效%20%2F%20162%20总计-059669?style=flat-square)](#九层根因分布-l0l8)
[![分类体系](https://img.shields.io/badge/分类体系-L0--L8%20九层模型-7c3aed?style=flat-square)](#九层根因分布-l0l8)
[![时间跨度](https://img.shields.io/badge/时间跨度-2022--2026-475569?style=flat-square)](#核心实证发现)

[English](./README.md) | **简体中文**

[核心统计报告](#核心统计报告) • [核心发现](#核心实证发现) • [九层根因体系](#九层根因分布-l0l8) • [13 仓台账导航](#分仓库台账导航) • [字段规范](#台账字段规范) • [贡献指南](#贡献指南)

</div>

---

## 项目简介

在现代 AI 基础设施（大模型推理引擎、GPU/NPU 算子库、深度学习编译框架与端侧引擎）中，**「单算子微基准显著提速，但集成到完整模型训练或在线推理后零收益、被严重稀释甚至性能倒退」**是高频出现的系统级难题。

本项目系统收集并整理了 **2022–2026 年间 13 个主流开源 AI Infra 仓库**中的 **162 条真实 GitHub Issue / PR 与官方技术博客复盘**（其中 **152 例为具备性能归因的 `L0–L8` 有效性能案例**，另含 10 例边界/非性能对照记录）。所有案例统一采用 **9 层根因分类体系（`L0–L8`）** 归档，每条记录均附可点击原帖超链接、软硬件负载、单算子与端到端双侧定量数据、Profiling 定位证据及社区最终解法。

---

## 核心统计报告

| 文档名称 | 格式 | 内容说明 |
| :--- | :---: | :--- |
| [**单算子到端到端性能鸿沟实证统计报告.md**](./reports/单算子到端到端性能鸿沟实证统计报告.md) | Markdown | 全库 152 例有效性能案例的完整多维统计报告（`L0–L8` 占比、13 仓分布矩阵、四大软件栈画像、复合耦合对、硬件平台、年度趋势、处置状态与九层工程防御清单） |
| [**单算子到端到端性能鸿沟实证统计报告.pdf**](./reports/单算子到端到端性能鸿沟实证统计报告.pdf) | PDF（4 页 A4） | 含 5 组高清矢量统计图（分布图、热力矩阵、软件栈堆叠图、耦合对与硬件图、年度与状态双联图）的排版报告 |

---

## 核心实证发现

- **核外系统税主导性能鸿沟（65.1%）**：真正发生在核函数内部的微架构退化（`L6`，11.2%）与硬件资源争抢（`L7`，5.3%）合计仅占 **16.4%**；相反，**胶水搬运/布局/反量化（`L3`，21.1%）、路由死路径与静默回退（`L1`，15.1%）、图编译/JIT/缓存生命周期反噬（`L4`，14.5%）与 Host 发射底噪（`L5`，14.5%）** 四大核外系统层合计高达 **65.1%**（99/152 例）。
- **绝大多数回归呈“多因耦合”特征（78.9%）**：152 例中共有 **120 例**涉及两个及以上层级的复合根因，平均每例涉及 **1.87 个**层级。最高频耦合对为 **`L3 + L5`**（17 例：布局/量化胶水叠加发射底噪）、**`L4 + L5`**（14 例：动态缓存键失效或 JIT 重编引发 Host 阻塞）与 **`L1 + L6`**（11 例：错误路由分派至微架构失配核）。
- **新一代硬件放大了外围系统税**：**2026 年占有效案例 68.4%（104 例）**，涵盖 NVIDIA Blackwell（33 次）、Hopper（28 次）、华为昇腾 Ascend（19 次）、AMD ROCm（15 次）与 Apple Silicon（11 次）。算力峰值越高、单核耗时越短，外围 Host 发射、JIT 重编与 Padding 拷贝越容易反噬收益。
- **`L6` 微架构失配最易触发正式回退（Revert）**：全库 **13 起正式回退（8.6%）** 中，`L6` 独占 **5 起（38.5%）**；全库仍有 **21.1%（32 例）** 处于 `Open` 状态，有待跨层架构与自动门控演进。

---

## 九层根因分布（`L0–L8`）

| 代码 | 根因类别名称 | 主根因案例数 | 主根因占比 | 含复合提及数 | 复合放大比 | 典型核心机理 |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- |
| **L0** | **评测失真与代理偏差** | 9 | 5.9% | 13 | 1.44x | Hot-L2 循环测速、合成均匀输入、墙钟混入包装器、CUPTI 探针放大、单槽/离散键盲区 |
| **L1** | **路由死路径与静默回退** | 23 | 15.1% | 33 | 1.43x | 黑名单静默拦截（0% 命中）、未注册 Silent No-Op、ABI 异常被吞、回退 SIMT/CPU、混合批一票否决 |
| **L2** | **关键路径稀释与阶段掩盖** | 16 | 10.5% | 27 | 1.69x | Amdahl 低占比稀释、异步流被主干 GEMM/通信掩盖、前反向阶段不对称、局部提速非可加性 |
| **L3** | **胶水搬运、布局转换与反量化** | **32** | **21.1%** | **59** | 1.84x | `.contiguous()` 全池拷贝、Head/DP/Page 零填充、NHWC↔NCHW 重排、未融合在线量化/反量化、H2D 搬运 |
| **L4** | **图编译、JIT 重编与缓存生命周期反噬** | 22 | 14.5% | 34 | 1.55x | Graph Break、Inductor FP64 隐式提升、TVM 激进内联、每步 JIT 重编/GIL 阻塞、Autotune 超时与图显存税 |
| **L5** | **Host 发射底噪与小 Shape 启动下限** | 22 | 14.5% | 51 | 2.32x | Python/FFI/CustomOp 分发税、每步元数据重算、同步气泡、碎裂 Peer-Copy 发射风暴、投机解码单步下限 |
| **L6** | **核内微架构退化** | 17 | 11.2% | 43 | **2.53x** | 寄存器溢出/Occupancy 悬崖、Wave 量化与尾块、跨代/异构（CDNA/RDNA/SXM/PCIe/Ascend）Tiling 失配、指令调度 |
| **L7** | **跨核资源争抢、缓存污染与功耗/频率反噬** | 8 | 5.3% | 17 | 2.12x | 共享矩阵引擎争抢（M4 SME）、并发流 SM/Workspace 竞态、L1/L2 局部性崩塌、功耗墙降频与 P-state 回落 |
| **L8** | **跨层数值误差放大与精度/接受率雪崩** | 3 | 2.0% | 7 | 2.33x | 深层舍入误差累积、TF32 截断致接受率暴跌、跨请求 Batch 非不变性、精度对齐强升 FP32 致带宽翻倍 |
| **合计** | **有效性能案例总计（主根因互斥）** | **152** | **100.0%** | **284** | **1.87x** | *另含 10 条边界/非性能对照记录（全库共 162 条）* |

---

## 分仓库台账导航

[`cases/`](./cases/) 目录下的每个 `.xlsx` 工作簿均包含 **3 个标准化工作表**：`案例总表`（15 列结构化台账，含可点击 GitHub 超链接）、`统计看板`（动态公式统计表与柱状图）以及 `根因分类字典`（`L0–L8` 分类说明）。其中 `vllm_metal_kernel_vs_e2e_cases.xlsx` 额外包含合并 `vLLM` 与 `vllm-metal` 的 `跨仓全景合并表`。

| 软件栈层级 | 开源仓库 | 台账文件 | 总条目 | 有效性能例（`L0–L8`） | 主导根因 |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **大模型推理引擎** | **TensorRT-LLM** | [tensorrt_llm_kernel_vs_e2e_cases_master.xlsx](./cases/tensorrt_llm_kernel_vs_e2e_cases_master.xlsx) | 18 | 18 | `L3` (6), `L5` (4), `L2` (3) |
| | **vllm-ascend** | [vllm_ascend_kernel_vs_e2e_cases_master.xlsx](./cases/vllm_ascend_kernel_vs_e2e_cases_master.xlsx) | 17 | 17 | `L3` (5), `L5` (4), `L6` (3) |
| | **vLLM** | [vllm_kernel_vs_e2e_cases_master.xlsx](./cases/vllm_kernel_vs_e2e_cases_master.xlsx) | 16 | 16 | `L3` (4), `L1` (3), `L5` (3) |
| | **SGLang** | [sglang_kernel_vs_e2e_cases_master.xlsx](./cases/sglang_kernel_vs_e2e_cases_master.xlsx) | 15 | 15 | `L0` (3), `L1` (3), `L2/L3/L5/L6` (2) |
| **算子库** | **FlashAttention** | [flash_attention_kernel_vs_e2e_cases_master.xlsx](./cases/flash_attention_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L1` (3), `L6` (3), `L4` (2) |
| | **FlashInfer** | [flashinfer_kernel_vs_e2e_cases_master.xlsx](./cases/flashinfer_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L4` (3), `L0` (2) |
| | **FlagGems** | [flaggems_kernel_vs_e2e_cases_master.xlsx](./cases/flaggems_kernel_vs_e2e_cases_master.xlsx) | 9 | 9 | `L1` (2), `L2` (2), `L5` (2) |
| | **Liger-Kernel** | [liger_kernel_vs_e2e_cases_master.xlsx](./cases/liger_kernel_vs_e2e_cases_master.xlsx) | 11 | 9 | `L4` (3), `L3` (2), `L5` (2) |
| **推理/编译框架** | **PyTorch** | [pytorch_kernel_vs_e2e_cases_master.xlsx](./cases/pytorch_kernel_vs_e2e_cases_master.xlsx) | 14 | 14 | `L4` (3), `L1/L3/L6` (2) |
| | **ONNX Runtime** | [onnxruntime_kernel_vs_e2e_cases_master.xlsx](./cases/onnxruntime_kernel_vs_e2e_cases_master.xlsx) | 10 | 10 | `L3` (6), `L1` (2), `L7` (2) |
| | **Apache TVM** | [tvm_operator_vs_e2e_cases_master.xlsx](./cases/tvm_operator_vs_e2e_cases_master.xlsx) | 13 | 6 | `L4` (4), `L1` (2) |
| **端侧/本地引擎** | **llama.cpp** | [llama_cpp_kernel_vs_e2e_cases_master.xlsx](./cases/llama_cpp_kernel_vs_e2e_cases_master.xlsx) | 16 | 16 | `L6` (6), `L3` (3), `L1/L5/L7` (2) |
| | **vllm-metal** | [vllm_metal_kernel_vs_e2e_cases.xlsx](./cases/vllm_metal_kernel_vs_e2e_cases.xlsx) | 5 | 4 | `L2` (3), `L5` (1) |

---

## 台账字段规范

全部 13 个工作簿的 `案例总表` 均采用统一的 15 列结构（`A:O`）：

1. `编号`（案例序号）
2. `Issue/PR`（可点击 GitHub 原帖链接）
3. `关联 PR / 追踪单`
4. `状态归类`（`Closed` / `Mitigated` / `Open` / `Reverted` / `Tech Blog`）
5. `详细状态与版本`
6. `年份` 与 `年月`
7. `优化算子 / 特性`
8. `硬件与 Workload`
9. `单算子 / 理论预期表现 (定量)`
10. `端到端 (E2E) 实测表现 (定量)`
11. `主根因` 与 `复合根因`（`L0–L8`）
12. `原帖 Profiling 定位证据`
13. `最终处置与社区解法`

---

## 贡献指南

欢迎通过 Issue 或 PR 补充更多真实工程案例！为保证实证严谨性，新增条目建议包含：

1. **可追溯来源**：公开的 GitHub Issue、PR、Commit 或官方工程博客链接。
2. **双侧定量证据**：同时给出（a）单算子微基准提速或明确理论优化预期，以及（b）完整模型或在线 Serving 端到端零收益、被稀释或性能倒退的实测数据。
3. **Profiling 依据与分类**：基于原帖 Profiler / 硬件计数器 / 源码归因标注 `L0–L8` 主根因与复合根因，不作臆测。
