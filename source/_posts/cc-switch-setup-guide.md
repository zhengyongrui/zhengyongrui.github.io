---
title: CC Switch 实战
date: 2026-06-05 10:00:00
tags:
  - CC Switch
  - Claude Code
  - OpenCode
  - AI编程
  - 教程
categories:
  - 开发工具
---

> 作者：zyr
> 日期：2026-06-05
> 适用对象：想在 Windows 环境下使用 CC Switch 统一管理多个 AI 模型供应商的开发者

---

## 前言

记录一次把 CC Switch + MiniMax M3 + OpenCode 全部配通的完整过程，含踩坑和修复。CC Switch 是 Windows 上最方便的 Claude Code / Codex / OpenCode 多供应商切换工具，本文从环境与工具介绍、配置项含义、模型路由、OpenCode 集成到常见踩坑逐一展开，供有需要的同学参考。

**背景**：项目需要同时对接 MiniMax、DeepSeek 等多家模型厂商，通过 CC Switch 统一环境变量，配合 OpenCode 完成多 Agent 协同开发。

---

## 一、环境与工具

| 工具 | 用途 |
|------|------|
| **CC Switch v3.16.0** | 多厂商模型路由，一键切换 Claude Code / Codex / OpenCode 的 API 供应商 |
| **MiniMax** | 国产大模型，M3 是最新旗舰（1M 上下文、原生多模态、SWE-Bench Pro 59%） |
| **OpenCode** | 开源 AI 编程 Agent，支持自定义 agent 分配不同模型 |
| **Claude Code** | Anthropic 官方 CLI，配合 CC Switch 使用第三方模型 |

---

## 二、配置项含义

CC Switch 通过环境变量把 Claude Code 内部对不同能力等级的模型调用映射到你指定的模型：

| 字段 | 含义 | 触发场景 |
|------|------|----------|
| `ANTHROPIC_MODEL` | 默认模型 | 启动对话和 `/model` 切换 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 轻量模型（原 Claude Haiku） | 简单补全、摘要、小修改 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 平衡型（原 Claude Sonnet） | 日常编码主力 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 旗舰模型（原 Claude Opus） | 复杂推理、大规模重构 |
| `ANTHROPIC_REASONING_MODEL` | 推理模型 | 数学/逻辑/算法等需要长链思考的任务 |

---

## 三、MiniMax M3 全系配置

MiniMax 于 2026-06-01 发布 M3，采用 MSA 稀疏注意力架构，1M 上下文，解码速度比 M2 快 15 倍。不想用旧模型，全部指向 M3：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-你的Key",
    "ANTHROPIC_BASE_URL": "https://api.minimaxi.com/anthropic",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "MiniMax-M3",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "MiniMax-M3",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "MiniMax-M3",
    "ANTHROPIC_MODEL": "MiniMax-M3",
    "ANTHROPIC_REASONING_MODEL": "MiniMax-M3",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1,
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

> 如果后续 MiniMax 出了 M3-highspeed 或新一代推理模型，可以把 Haiku 指向高速版、Reasoning 指向推理版，实现真正的分级路由。

---

## 四、MiniMax 与 DeepSeek 能否混用？

**不能在同一套配置里混用。** CC Switch 每个供应商配置是整体切换的——`BASE_URL` 和 `AUTH_TOKEN` 都是供应商级别，不是模型级别。

- 想切供应商：系统托盘一键切换，轻任务用便宜的，复杂任务切 M3
- 想自动化路由：等 CC Switch 或模型厂商支持按模型粒度配置多组 API 端点

---

## 五、OpenCode 集成

### 5.1 给 plan / build / Superpowers 分配不同模型

opencode 通过 `agent` 字段（`mode` 已弃用）为不同模式指定模型：

```json
{
  "model": "minimax/MiniMax-M3",
  "small_model": "minimax/MiniMax-M3",
  "agent": {
    "plan": {
      "model": "minimax/MiniMax-M3",
      "temperature": 0.1
    },
    "build": {
      "model": "minimax/MiniMax-M3"
    }
  },
  "plugin": [
    "superpowers@git+https://github.com/obra/superpowers.git"
  ]
}
```

目前全用 M3 即可，后续按需拆分。

### 5.2 CC Switch 导入报错：opencode.json 解析失败

**现象**：CC Switch 启动日志报 `JSON 解析错误: opencode.json: EOF while parsing a value`

**原因**：`~/.config/opencode/opencode.json` 是空文件。

**修复**：写入 `{}` 即可。

---

## 六、实用 MCP 服务推荐

| MCP 服务 | 安装命令 | 用途 |
|----------|----------|------|
| **Context7** | `npx ctx7 setup --claude` 或 `--opencode` | 实时拉取最新版本文档，告别训练数据过时 |
| **Sequential Thinking** | `claude mcp add sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking` | 强制分步推理，复杂逻辑不跳步 |
| **Playwright** | `claude mcp add playwright -- npx @playwright/mcp@latest` | 浏览器自动化，不用写测试脚本 |
| **GitHub** | `claude mcp add-json github '{...}'` | PR/Issue/代码搜索，不切浏览器 |
| **Memory** | `claude mcp add memory -- npx -y @modelcontextprotocol/server-memory` | 跨会话记忆 |

### 验证安装

```bash
# Claude Code
claude mcp list

# OpenCode
opencode mcp list
```

或在对话中输入 `/mcp`。

---

## 七、踩坑记录

### 1. Fetch MCP 折腾三部曲

| 尝试 | 配置 | 结果 |
|------|------|------|
| 第 1 次 | `command: uvx`, `args: mcp-server-fetch` | `✘ failed`（Windows 没装 uvx） |
| 第 2 次 | `command: cmd`, `npx @modelcontextprotocol/server-fetch` | `E404`（npm 上不存在） |
| 第 3 次 | `command: cmd`, `npx mcp-server-fetch` | 能跑但 -32000 报错（是安全蜜罐假包） |
| **最终** | `command: cmd`, `npx -y @kazuph/mcp-fetch` | `✔ connected` |

**教训**：
- Windows 上优先用 `cmd /c npx`，别碰 `uvx`
- npm 包名以实际仓库为准，`@modelcontextprotocol/server-fetch` 在新 monorepo 中已不在 npm 托管
- `mcp-server-fetch` 是蜜罐包，安装前务必 `npm view` 确认

### 2. JSON 尾随逗号

```json
// ❌ 错误
{ "$schema": "...", }

// ✅ 正确
{ "$schema": "..." }
```

### 3. CC Switch 更新

便携版更新：从 [GitHub Releases](https://github.com/farion1231/cc-switch/releases) 下载最新 zip，解压覆盖 `cc-switch.exe`。数据在 `~/.cc-switch/` 不受影响。

---

## 八、最终效果

- **Claude Code**：通过 CC Switch 接入 MiniMax M3，所有能力等级统一映射
- **OpenCode**：独立配置，plan/build/Superpowers 各司其职
- **MCP**：Context7（文档实时）+ Sequential Thinking（深度推理）+ Memory（跨会话记忆）+ Fetch（网页抓取）
- **热切换**：系统托盘一键在 MiniMax / DeepSeek 之间切换

整个链路打通后，日常编码用 M3 主力扛，复杂任务自动调推理链路，文档永远是最新版，不用再手写 `settings.json`，也不用在多个配置文件之间反复横跳。

---

## 九、总结

本次调优的核心要点：

1. **供应商维度切换**：CC Switch 一套环境变量绑定一个供应商，BASE_URL + AUTH_TOKEN 必须匹配，不能混用
2. **能力等级映射**：Haiku / Sonnet / Opus / Reasoning 全部指向 M3，实现统一升级，未来按需拆分
3. **OpenCode 独立配置**：plan / build / Superpowers 通过 `agent` 字段分配模型，与 Claude Code 解耦
4. **MCP 服务精选**：Context7（实时文档）、Sequential Thinking（深度推理）、Memory（跨会话）是性价比最高的三个
5. **Windows 兼容**：MCP 服务优先用 `cmd /c npx -y`，避免 uvx 和蜜罐包
6. **多多利用AI工具**：全程使用腾讯最新的Marvis配置的，这个工具目前还是太慢了

CC Switch 把多供应商管理这件原本很麻烦的事做成了"系统托盘一点切换"，配合 OpenCode 的多 Agent 能力，是目前 Windows 下最顺滑的 AI 编码工作流之一。

---

*文档生成时间：2026-06-05*
*工具版本：CC Switch v3.16.0*
