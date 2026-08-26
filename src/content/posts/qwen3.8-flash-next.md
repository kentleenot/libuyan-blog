---
title: "Qwen3.8-Flash-Next 发布：通往 Qwen4 的架构预览"
author: "李不言"
pubDatetime: 2026-08-26T23:50:00+08:00
slug: qwen3.8-flash-next
featured: false
draft: false
tags:
  - AI
  - 大模型
  - Qwen
description: "阿里 8 月 26 日开源 Qwen3.8-Flash-Next：125B MoE 模型、每 token 仅激活 6B 参数，训练成本约为 Qwen3.7-Plus 的九分之一，却首次公开了 Qwen4 的核心架构——GDN+QSA 混合注意力、门控残差、N-gram 嵌入与 Muon 优化器。"
---

**核心结论：Qwen3.8-Flash-Next 不是又一个小版本升级，而是阿里提前公开的 Qwen4 架构预览。125B 总参数的 MoE 模型，每 token 只激活 6B，训练成本约为 Qwen3.7-Plus 的 1/9，编码与办公任务能力反而更强。它把「稀疏激活 + 状态压缩 + 嵌入查表」三条降本路线全部压进了一个开源模型。**

2026 年 8 月 26 日，Qwen 团队正式开源 Qwen3.8-Flash-Next（Hugging Face 模型 ID：`Qwen/Qwen3.8-Flash-Next`，GitHub 仓库：`QwenLM/Qwen3.8-Flash-Next`）。官方明确定位：这是一个多模态 MoE 模型，同时也是 Qwen4 所用架构的早期预览——就像当年的 Qwen3-Next 预告了 Qwen3.5 系列一样。Gated DeltaNet + Gated Attention 这套混合设计，后来被 Qwen3.5、3.6、3.7、3.8 全系列沿用；而这次公开的架构改动，将在完整的 Qwen4 家族上继续演进。

## 一、这到底是什么：一张表看懂规模

| 项目 | 数值 |
|------|------|
| 总参数量 | 125B 主模型 + 51B N-gram 嵌入表 |
| 每 token 激活参数 | 6B |
| 训练成本（对比 Qwen3.7-Plus） | 约 1/9 |
| 上下文长度 | 262,144（官方 SGLang 示例） |
| 发布形态 | 多模态 MoE，开源权重（Hugging Face / ModelScope） |
| 定位 | Qwen4 架构早期预览 |

「125B 参数只激活 6B」意味着什么？推理时绝大多数参数不参与计算，显存压力与算力消耗接近一个 6B 稠密模型，但能力上限由 125B 的总知识容量兜底。这就是 MoE 路线的核心逻辑：**用稀疏激活换能力，用总参数量换知识密度。**

## 二、四大架构升级：每个都冲着成本去

官方把这代升级归纳为四个维度：注意力、残差、嵌入、优化。逐一拆开看，全部指向同一件事——**用更少的计算量实现更强的能力**。

### 1. Attention：GDN + QSA 混合架构

注意力是长上下文成本的大头。Qwen3.8-Flash-Next 用两条腿走路：

- **Gated DeltaNet（GDN）**：一种带门控的线性注意力变体，用固定大小的状态高效压缩历史信息，让「记忆」不随序列长度线性膨胀。
- **Qwen Sparse Attention（QSA）**：用一个轻量压缩索引器，在微块（micro-block）粒度挑选重要上下文，跳过无关内容，大幅降低长序列上的注意力计算量。

一句话：**能压缩的压缩，能跳过的跳过。** 这正是 262K 长上下文能跑起来的成本前提。

### 2. Residual：门控残差（Gated Residual）

把残差流从单路拓宽到 4 个分支，并用动态门控控制读写。跨层信息流更强、训练更稳定。残差结构这种「地基级」改动，通常不会立刻反映在 benchmark 上，但它决定了模型能不能稳定地训练到足够大——这是 Qwen4 的基建投资。

### 3. Embedding：N-gram 嵌入表

这是最「反常规」的一招：**用局部上下文查表**。N-gram Embedding 把一部分知识从可学习的神经网络参数里，挪进一张离线嵌入表——查表几乎不花算力，却能显著扩展模型容量。更妙的是，这张表可以卸载到主机内存，通过异步预取与模型计算重叠，GPU 显存压力进一步降低。

本质上这是「知识存储与计算解耦」：计算密集的部分留在 GPU 上，存储密集的部分放内存里。

### 4. Optimization：Muon 优化器

训练侧改用 Muon 优化器，并针对正交化精度、Muon 与 AdamW 的分工、融合参数拆分做了精调，同时按新架构重新拟合了缩放定律。训练成本能压到 1/9，优化器与缩放定律的重新校准功不可没。

## 三、效率与能力的剪刀差

官方给出的核心卖点：**训练成本约为 Qwen3.7-Plus 的 1/9，编码与办公任务能力反而更强。**

这个「剪刀差」是这代模型最值得关注的地方。大模型行业过去两年的主旋律是「训练成本指数级上升」，而 Qwen3.8-Flash-Next 展示的是另一条路：**架构级优化把成本曲线往下压，而不是靠堆算力硬扛。**

对普通用户和开发者来说，这意味着更低的 API 价格空间、更强的端侧部署可能性，以及——如果这套架构成为 Qwen4 的基础——未来两年 Qwen 系模型的成本结构会与今天完全不同。

## 四、生态与部署：从云端到本地全线铺开

- **QwenWork**：新发布的「Standard」模式已由 Qwen3.8-Flash-Next 驱动，QwenWork 是阿里的 AI 办公平台。
- **QwenCloud API**：兼容 OpenAI 与 Anthropic 两套 API 规范，接入成本低。
- **本地推理**：llama.cpp（支持文本与视觉）、Unsloth（量化微调）、Transformers 的 `serve` 命令一条命令起服务。
- **服务化部署**：SGLang、vLLM、TokenSpeed 三大框架均已支持，官方示例统一使用 `--context-length 262144`，长上下文是默认配置。

本地跑法也很简单（以 transformers 为例）：

```shell
transformers serve Qwen/Qwen3.8-Flash-Next --port 8000 --continuous-batching
```

## 五、我的看法

第一，**「激活 6B / 总参 125B」的差距还会继续拉大。** 稀疏激活 + 状态压缩（GDN）+ 稀疏注意力（QSA）+ 查表嵌入（N-gram），四条技术路线全部指向同一个方向：把「模型的容量」和「推理的成本」彻底解耦。这是大模型走向普及的必经之路。

第二，**开源节奏在加速，且越来越「预告式」。** 把 Qwen4 的架构提前一个版本放出来给社区检验，既能收集生态反馈，也让第三方框架提前适配。等到 Qwen4 正式发布时，部署生态已经是现成的。这是很聪明的产品策略。

第三，**别只盯着 benchmark，盯成本结构。** 1/9 的训练成本如果延续到 Qwen4 全系，受影响的不只是阿里——整个推理服务市场的定价锚都会跟着下移。

需要留意的是，官方完整 benchmark 表格发布在 qwen.ai 博客（JS 渲染页面，抓取不便），本文数据以官方 GitHub README 与 Hugging Face 模型卡为准。架构细节论文为《On the Design of Qwen3.8-Next Architecture: Evaluation, Efficiency, and Training Stability》（Qwen Team, Alibaba Group, 2026-08）。

---

**参考来源：**
- 官方仓库：[QwenLM/Qwen3.8-Flash-Next](https://github.com/QwenLM/Qwen3.8-Flash-Next)
- 官方博客：[Qwen3.8-Flash-Next: A New Architecture, Towards Ultimate Cost-Efficiency](https://qwen.ai/blog?id=qwen3.8-flash-next)
- 模型权重：[Qwen/Qwen3.8-Flash-Next on Hugging Face](https://huggingface.co/Qwen/Qwen3.8-Flash-Next)
- 架构图：qianwen-res.oss-accelerate.aliyuncs.com（已本地化）
