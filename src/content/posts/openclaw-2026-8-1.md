---
title: "OpenClaw 2026.8.1：个人 AI 助手的“2.0 时刻”"
author: "李不言"
pubDatetime: 2026-08-31T23:35:00+08:00
slug: openclaw-2026-8-1
featured: false
draft: false
tags:
  - AI
  - OpenClaw
  - 开源
  - 工具
description: "OpenClaw 于 8 月末发布 2026.8.1 大版本：会话可搜索、可跨设备与云端迁移，新增持久进度卡、结构化问答与交互式仪表盘，凭据请求与审批机制把安全边界做实，同时带来两项 breaking change 与官方 Provider 拆分。本文基于官方 CHANGELOG 与本机升级实测。"
---

**核心结论：OpenClaw 采用日期制版本号，2026 年 8 月末发布的 2026.8.1 是近期体量最大的一次更新，完全配得上“2.0 时刻”的称呼。这一次它从“单机助手”走向“个人 AI 基础设施”：历史会话可以全文搜索、工作区可以跟着会话迁移到配对设备或云端；持久进度卡、结构化问答、交互式仪表盘把“看得见 agent 在做什么”变成默认体验；私有凭据请求、一次性审批、共享凭据库把安全边界做实；同时伴随两项 breaking change（OpenProse 移除、OpenAI 路由迁移）和一批官方 Provider 拆分安装。本文基于官方 CHANGELOG 与本机升级实测（2026.7.1-2 → 2026.8.1）。**

先说清楚版本号：OpenClaw 的版本号是日期制的（2026.8.1 = 2026 年 8 月第 1 个稳定版），官方并没有“2.0”这个版本号——GitHub Releases 最新 tag 是 `v2026.8.1`，npm 最新稳定版同样是 2026.8.1，beta 线已经走到 2026.9.1-beta.1。所以本文标题里的“2.0”，指的是这次更新在能力上的代际跃迁，而不是官方版本命名。

## 一、OpenClaw 是什么

OpenClaw 是一个开源 AI 助手运行时：一个 Gateway 把模型、工具、消息渠道连在一起，你在微信、Discord、Slack、Telegram 里直接跟 agent 说话，它在本机或云端执行任务。同一套架构既能跑在个人笔记本上，也能部署成团队共享服务——配置是唯一的区别。它的架构主张是“可信的 Gateway、不可信的执行、确定性的策略”，即模型只能通过 Gateway 授权的工具与世界交互。

## 二、2026.8.1 亮点一览

| 维度 | 能力 | 一句话 |
|------|------|--------|
| 会话 | 历史全文搜索 | 按词或短语搜过去的对话，直接重开上下文 |
| 会话 | 跨设备/云会话 | 工作区随会话迁移，复用热机器与项目种子 |
| 交互 | 持久进度卡 | reload 不丢，web 与原生端跟随子代理与编辑 |
| 交互 | 结构化问答 | 卡片/按钮/纯文本三种方式回答，可自由输入或跳过 |
| 交互 | 交互式仪表盘 | 聊天内 widget 钉到会话面板，可导出图片 |
| 安全 | 私有凭据请求 | 掩码输入，值不进聊天与模型上下文 |
| 安全 | 一次性审批 | 自动化精确操作授权一次，可查可撤销 |
| 协作 | 团队角色与共享凭据库 | 限定可访问的 agent 与会话，SQLite 凭据 write-only |
| 智能 | Grounded dreaming | 模型驱动的背景记忆整理，默认开启 |
| 智能 | 自动学习 | 捕获强经验并自动应用合规技能 |

## 三、值得展开的四个亮点

**第一，会话搜索 + 跨设备会话：工作区跟着会话走。** 过去 agent 的历史只能翻会话记录，现在可以直接按词搜索并重开上下文；更关键的是会话可以“搬家”——在配对设备或云 worker 上继续跑，工作区随之迁移，还能复用之前暖好的机器。对于有多台设备的用户，这意味着“在家开的头，在办公室接着干”不再需要手动同步。

**第二，持久进度卡与结构化问答：协作体验质变。** 新版本把“进度”做成了会话的一部分：一个进度卡跨 reload 存活，web、macOS、iOS、Android 都能看到最新状态；agent 提问时以结构化卡片呈现，你点按钮、回自由文本或跳过都行。这对长任务（比如跑实验、写论文、部署服务）是实打实的体验提升——不用再翻聊天记录猜它卡在哪。

**第三，私有凭据请求与一次性审批：安全默认值。** agent 需要 API Key 时，通过掩码提示请求，值只进受保护的凭据存储，不进聊天记录、不进模型上下文；还可以用代理把密钥替换限制在批准的目标域名内。同时自动化任务可以“一次授权”，任务或操作变化时需要重新审批。这两条组合起来，把“agent 能拿到什么、能花掉什么”变成了可审计、可撤销的。

**第四，Grounded dreaming 与自动学习：记忆与技能开始自进化。** 默认开启的模型驱动记忆整理会把带来源标注的材料沉淀进长期记忆（有 Dream Diary 可查、可关闭）；自动学习则把“这次踩坑学到的经验”固化成技能并自动应用。对重度用户来说，这意味着 agent 不只是会聊天，而是会积累——这正是“助手”与“伙伴”的分水岭。

## 四、升级必读：两项 breaking change

1. **OpenProse 迁移**：内置 OpenProse 插件与 `/prose` 命令被移除，需要 `openclaw doctor --fix` 清理旧配置，原 `.prose` 源文件保留，按官方迁移指南升级。
2. **OpenAI 路由迁移**：`codex/*` 与 `openai-codex/*` 模型引用、Provider 配置、历史会话与自动化路由统一迁移到 `openai/*`，同样用 `openclaw doctor --fix` 完成，冲突项会标记出来交人工修复。
3. **SDK 弃用预告（2026-09-01 起）**：外部插件的 `plugin-sdk-*` 子路径导入将迁移到聚焦的新导入路径，这是“预告门”而非本次直接移除，但插件作者需要提前迁移。
4. **官方 Provider 拆分**：BytePlus、ComfyUI、Mistral、NovitaAI、OpenCode、Synthetic、Volcengine、Vydra、Xiaomi 等改为按需安装的官方包；缺失的包可用 `openclaw update repair` 或 `openclaw doctor --fix` 恢复。

## 五、我的升级实测（2026.7.1-2 → 2026.8.1）

8 月 31 日晚本机完成升级，全程约 5 分钟，几个值得记录的坑：

- **npm 直连失败**：`npm install -g openclaw@latest` 直连 registry.npmjs.org 报 `ECONNRESET`（网络问题，非包问题），改用国内镜像 `--registry=https://registry.npmmirror.com` 一次成功。
- **doctor --fix 是关键一步**：新版本要求状态库 schema 迁移（audit-events-v2），同时清理了 openclaw-weixin 插件的重复安装记录——这两件事都靠 `openclaw doctor --fix` 完成。
- **Gateway 重启验证**：`openclaw gateway status` 确认 18789 端口正常监听，微信通道（openclaw-weixin）在兼容性检查 `[compat] Host OpenClaw 2026.8.1 >= 2026.3.22, OK` 后正常收发消息，模型请求（DeepSeek）响应正常。

升级动作本身很简单：`npm install -g openclaw@latest`（或官方安装脚本）→ `openclaw doctor --fix` → 重启 Gateway。官方建议升级前备份配置与状态，本机 `openclaw backup` 也支持 SQLite 快照。

## 六、我的看法

OpenClaw 正在从“单机助手”变成“个人 AI 基础设施”。判断依据有三：会话可以搜索和迁移（数据资产化）、凭据可以安全托管（权限资产化）、技能可以自动沉淀（经验资产化）——三样凑齐，它就不再是个聊天框，而是你的第二台大脑主机。

版本节奏明显加快：8 月一个月就有 7.1-2 与 8.1 两个稳定版，beta 线已到 9.1。升级时要跟着 doctor 走，breaking change 都提供了迁移路径，但建议升级前看一眼 CHANGELOG 再动手。

下一步值得盯的三件事：A2A 1.0 通道（agent 之间互发任务的互操作标准）、云 worker 生命周期（闲置挂起、按需拉起）、桌面控制（macOS 默认开启，Windows/Linux 仍是实验性）。多设备、多 agent 的拼图正在一块块补齐。

需要说明的局限：本文亮点与 breaking change 信息以官方 CHANGELOG（2026.8.1）为准，未逐项做功能级验证；版本日期制命名下，“2.0”是我对本次更新体量的个人概括，不是官方版本号。

---

**参考来源：**
- 官方 Releases：[openclaw/openclaw on GitHub](https://github.com/openclaw/openclaw/releases)
- npm 包：[openclaw](https://www.npmjs.com/package/openclaw)
- 官方文档：[docs.openclaw.ai](https://docs.openclaw.ai)
