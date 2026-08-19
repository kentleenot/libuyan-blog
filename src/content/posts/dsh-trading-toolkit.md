---
title: "dsh-trading-toolkit：给 DSH 智能体的 A股+美股行情工具箱"
author: "李不言"
pubDatetime: 2026-08-18T12:30:00+08:00
slug: dsh-trading-toolkit
featured: false
draft: false
tags:
  - DeepSeek Harness
  - 插件
  - 开源
  - A股
  - 美股
description: "A-share and US stock market data tools for DeepSeek Harness — 实时行情 · K线 · 市场状态信号 · 回测预览。只读，免 Key，双市场。"
---

> A-share and US stock market data tools for DeepSeek Harness — 实时行情 · K线 · 市场状态信号 · 回测预览。只读，免 Key，双市场。

🏆 **收录 5/7** · [awesome-deepseek-harness](https://github.com/Dominic789654/awesome-deepseek-harness)（Dominic #130 ✅）· [awesome-dsh-plugin](https://github.com/Anil-matcha/awesome-dsh-plugin)（Anil-matcha #28 ✅）· [awesome-deepseek-harness](https://github.com/0xsline/awesome-deepseek-harness)（0xsline #381 ✅）· [Awesome-DeepSeek-Harness-Plugins](https://github.com/Zhiyuan-Fan/Awesome-DeepSeek-Harness-Plugins)（Zhiyuan-Fan #33 ✅）· [awesome-dsh-plugins](https://github.com/AdamPlatin123/awesome-dsh-plugins)（AdamPlatin123 #243 ✅ cherry-pick）· 审核中 2：awesome-dsh-plugin #1551、libukai #40 · 🏷️ `dsh-plugin` topic · 📄 MIT License

> ⚠️ **免责声明：** 本插件及本文内容仅供技术研究与学习交流，不构成任何投资建议。投资有风险，入市需谨慎。

---

## 为什么做

DSH 智能体很强，但有个尴尬：让它查 A股行情，它只能去 `web_search` 抓 HTML 页面，一次抓一行，慢且不可靠。没有市场数据工具，agent 只能 improvise。

dsh-trading-toolkit 给智能体四个一等公民工具，让它不再对着网页瞎猜。

## Before / After

同一个任务："查一下贵州茅台的现价和最近 20 天 K线趋势"，同一模型，唯一变量是插件装没装：

| | 无插件 | 有插件 |
|---|---|---|
| 工具调用 | 反复 web_search + 解析 HTML | 1 次调用 |
| 数据可靠性 | 页面结构变了就废 | 结构化 JSON，字段固定 |
| 耗时 | 数秒~数十秒（依赖网页加载） | 毫秒级（东财 push2 接口） |
| 覆盖 | 只能碰运气搜到的页面 | 沪深 5000+ + 美股全量 |

实测（2026-08-18）：`get_kline` 茅台收 1291.83、NVDA 收 $225.01，A股美股双向一次通过。

## 四个工具

| 工具 | 功能 |
|---|---|
| `get_quote` | 实时行情（沪深 + 美股），毫秒级报价 |
| `get_kline` | OHLCV K线：分钟到月线，前复权/后复权/不复权，beg/end 控制范围 |
| `get_signal` | ADX 三状态市场分类（趋势 / 震荡 / 噪声）——告诉 agent 当前该用什么策略思路 |
| `backtest_preview` | 策略想法先跑历史数据看效果，再决定是否实盘 |

## 设计理念

- **只读设计**：只查数据，永不下单——agent 权限再大也搞不出风险事件
- **免 Key 免代理**：数据源用东方财富，国内直连，不需要用户配任何 API 密钥
- **双市场一套接口**：同一套工具同时服务 A股和美股分析场景，schema 统一
- **纯标准库实现**：零重依赖，装完即用，不给用户环境添负担
- **AI-native**：工具 schema 从模型视角写，参数/输出契约明确，模型不用猜

## 技术要点

- 数据源：东方财富 push2 行情接口（免费、稳定、双市场）
- 美股代码探测：直接 probe K线 API 三市场（105/106/107），不依赖写死的映射表
- limit 处理：东财 `lmt` 参数被忽略，用 `beg/end` 日期范围控制行数 + 本地 slice
- 架构：`lib/market.js`（行情）+ `lib/kline.js`（K线）+ `lib/signal.js`（信号）+ `lib/index.js`（工具注册）

## 快速上手

```bash
dsh plugin --profile web add github:kentleenot/dsh-trading-toolkit
```

（`web` 是 profile 名，可换成你自己的 profile；安装命令经 dsh CLI 0.1.0-rc.7 实测验证。）

然后直接问你的 agent："查一下宁德时代现价，顺便看看它现在是趋势还是震荡。"

## 路线图

- [ ] 更多信号类型（RSI 背离、布林带反转）
- [ ] 板块/概念资金流
- [ ] 财报日历提醒

---

*工具是手段，理解是目的。让 agent 拿到数据，而不是拿到网页。*
