---
title: OpenCode 实战：从安装配置到 Superpowers 工作流的完整指南
date: 2026-05-03 10:00:00
tags:
  - OpenCode
  - AI编程
  - 微信小程序
  - 教程
categories:
  - 开发工具
---

> 作者：zyr
> 日期：2026-05-03
> 适用对象：想在 Windows 环境下使用 OpenCode 改进微信小程序项目的开发者

---

## 前言

今天花了一整天时间和 OpenCode 深度打交道，从零安装配置到跑通 Superpowers 多步工作流，把整个过程踩了一遍。这篇文档把学到的东西整理出来，供有需要的同学参考。

**背景**：项目是一个**智能取水系统 SaaS**（微信小程序 + Spring Boot 后端），需要分步骤改进前端交互、添加功能模块、完善管理后台。

---

## 一、OpenCode 简介

OpenCode 是一个基于终端的 AI 编程助手（类 Cursor AI CLI 版），支持：

- 多模型接入（OpenAI、Anthropic、Groq、OpenRouter 等）
- Agent 工作流（Superpowers 等多步协作 Agent）
- MCP（Model Context Protocol）扩展
- TUI 终端界面 + 命令行模式

官网：https://opencode.ai

---

## 二、安装与环境

### 2.1 安装方式

**推荐方式：使用 QClaw 安装**

QClaw 是目前最方便的 OpenCode 安装方式，一键完成安装和配置。

```powershell
# 通过 QClaw 安装 OpenCode
# 具体安装命令请参考 QClaw 官方文档
```

**备选方式：npm 直接安装**

```powershell
npm install -g opencode-ai
```

安装成功后，版本检查：

```powershell
opencode --version
# 输出示例：opencode v1.14.33
```

### 2.2 重要路径（Windows 环境）

| 用途 | 路径 |
|------|------|
| 全局配置目录 | ~/.config/opencode/ |
| 数据存储目录 | ~/.local/share/opencode/ |
| API 凭证文件 | ~/.local/share/opencode/auth.json |

### 2.3 启动方式

```powershell
# 默认启动（TUI 界面）
opencode

# 指定目录启动
opencode D:\WeChatProjects\ysj-project

# 直接跑命令（非交互模式）
opencode run "解释这个函数的用途"

# 继续上次的会话
opencode --continue
# 或
opencode -c
```

---

## 三、基础配置

### 3.1 配置 Coding Plan（阿里云百炼）

推荐使用阿里云百炼 Coding Plan，支持多种国产大模型。

**配置文件路径：**
- macOS / Linux: `~/.config/opencode/opencode.json`
- Windows: `C:\Users\您的用户名\.config\opencode\opencode.json`

**关键配置项说明：**
- **Base URL**：`https://coding.dashscope.aliyuncs.com/apps/anthropic/v1`（末尾必须带 `/v1`，否则报 404）
- **API Key**：格式为 `sk-sp-xxx`，从阿里云百炼控制台获取

**完整配置示例：**

```json
{
    "$schema": "https://opencode.ai/config.json",
    "provider": {
        "bailian-coding-plan": {
            "npm": "@ai-sdk/anthropic",
            "name": "Model Studio Coding Plan",
            "options": {
                "baseURL": "https://coding.dashscope.aliyuncs.com/apps/anthropic/v1",
                "apiKey": "YOUR_API_KEY"
            },
            "models": {
                "qwen3.6-plus": {
                    "name": "Qwen3.6 Plus",
                    "modalities": {
                        "input": ["text", "image"],
                        "output": ["text"]
                    },
                    "options": {
                        "thinking": {
                            "type": "enabled",
                            "budgetTokens": 8192
                        }
                    },
                    "limit": {
                        "context": 1000000,
                        "output": 65536
                    }
                },
                "qwen3-coder-plus": {
                    "name": "Qwen3 Coder Plus",
                    "modalities": {
                        "input": ["text"],
                        "output": ["text"]
                    },
                    "limit": {
                        "context": 1000000,
                        "output": 65536
                    }
                },
                "glm-5": {
                    "name": "GLM-5",
                    "modalities": {
                        "input": ["text"],
                        "output": ["text"]
                    },
                    "options": {
                        "thinking": {
                            "type": "enabled",
                            "budgetTokens": 8192
                        }
                    },
                    "limit": {
                        "context": 202752,
                        "output": 16384
                    }
                }
            }
        }
    }
}
```

**支持的模型列表：**

| 模型 | 上下文长度 | 输出长度 | 特性 |
|------|-----------|---------|------|
| qwen3.6-plus | 1M | 65K | 支持图像、思考模式 |
| qwen3.5-plus | 1M | 65K | 支持图像、思考模式 |
| qwen3-max | 256K | 32K | 通用大模型 |
| qwen3-coder-plus | 1M | 65K | 代码专用 |
| glm-5 | 200K | 16K | 支持思考模式 |
| kimi-k2.5 | 256K | 32K | 支持图像、思考模式 |

> 参考文档：https://bailian.console.aliyun.com/cn-beijing/?tab=doc#/doc/?type=model&url=3023086

### 3.2 配置 DeepSeek API

DeepSeek 是国内领先的 AI 模型提供商，OpenCode 支持通过 `/connect` 命令快速接入。

**前提条件：** 确保 OpenCode 版本 >= v1.14.24

**从现有安装迁移：**

1. 在输入框中输入 `/connect`，选择 `deepseek` 供应商

2. 填入 DeepSeek API Key

3. 选择 `DeepSeek-V4-Pro` 模型


> 参考文档：https://api-docs.deepseek.com/zh-cn/quick_start/agent_integrations/opencode

### 3.3 查看可用模型

```powershell
# 列出所有可用模型
opencode models

# 刷新模型列表
opencode models --refresh

# 详细模式（含价格）
opencode models --verbose
```

### 3.4 验证配置

保存配置后，重启 OpenCode 使配置生效：

```powershell
# 退出当前会话
/exit

# 重新启动
opencode

# 验证模型是否加载成功
opencode models
```

---

## 四、TUI 界面与快捷键

### 4.1 三大工作模式（Tab 切换）

在 TUI 底部按 **Tab** 键循环切换：

| 模式 | 说明 | 能改代码？ |
|------|------|-----------|
| **Plan** | 规划模式，只出方案不动手 | ❌ |
| **Build** | 构建模式，直接执行 | ✅ |
| **Superpowers** | 多步工作流，标准开发流程 | ✅ 分步确认 |

### 4.2 核心快捷键（Ctrl+X 前缀）

| 快捷键 | 作用 |
|--------|------|
| `Ctrl+X n` | 开新会话 |
| `Ctrl+X l` | 列出/切换历史会话 |
| `Ctrl+X c` | 压缩当前会话 |
| `Ctrl+X u` | 撤销上一条消息 |
| `Ctrl+X r` | 重做撤销的消息 |
| `Ctrl+X a` | 列出所有 Agent |
| `Ctrl+X m` | 查看模型列表 |
| `Ctrl+X e` | 用外部编辑器写消息 |
| `Ctrl+X x` | 导出会话为 Markdown |
| `Ctrl+X t` | 切换主题 |
| `Ctrl+X b` | 切换侧边栏 |
| `Ctrl+X s` | 查看状态 |
| `Ctrl+X q` | 退出 |

### 4.3 子 Agent 会话导航

Superpowers 运行时，按以下键在父子会话间切换：

| 快捷键 | 作用 |
|--------|------|
| `Ctrl+X ↓` | 跳到第一个子 Agent 会话 |
| `→` | 切换到下一个子 Agent |
| `←` | 切换到上一个子 Agent |
| `↑` | 返回父会话（主协调器） |

### 4.4 消息浏览与输入

| 快捷键 | 作用 |
|--------|------|
| `↑` / `↓` | 上下条消息 |
| `PageUp` / `PageDown` | 翻页 |
| `Ctrl+G` | 跳到最顶部 |
| `Ctrl+Alt+G` | 跳到最底部 |
| `Shift+Enter` | 换行（不发送） |
| `@文件名` | 引用文件内容 |
| `!命令` | 直接执行 shell 命令 |

### 4.5 命令面板

| 快捷键 | 作用 |
|--------|------|
| `Ctrl+P` | 打开命令面板，搜索所有命令 |
| `Esc` | 中断正在运行的响应 |

---

## 五、Superpowers 工作流

### 5.1 什么是 Superpowers

Superpowers 是一个多步 Agent 工作流，把开发过程拆成标准步骤：

```
头脑风暴 → 写规格文档 → 审计规格 → 写实现计划 → 执行编码
```

每个步骤需要用户确认后再继续，比直接让 AI 改代码更可控。

### 5.2 安装方式

Superpowers 需要单独安装，参考项目：https://github.com/mrth2/opencode-superpowers

**安装命令：**

```powershell
npx opencode-superpowers@latest
```

安装完成后，可通过以下方式启用：

```powershell
# 查看可用 Agent
opencode agent list

# 启动 OpenCode
opencode
```

进入 TUI 后，按 **Tab** 键切换到 **Superpowers** 模式即可使用。

### 5.3 使用方式

切换到 Superpowers 模式后，输入任务描述，例如：

```
帮我把水卡页面改造成角色自适应的管理页面
```

Superpowers 会自动：
1. 先进行头脑风暴（brainstorming）
2. 输出规格文档让你确认
3. 审计规格找问题
4. 生成实现计划
5. 逐任务执行编码

### 5.4 配置子 Agent 模型

Superpowers 包含多个子 Agent 协同工作，每个负责不同阶段。默认使用免费模型，如需切换更强模型，可手动修改配置。

**建议先查看可用模型列表：**

```powershell
opencode models
```

这样可以避免输入错误的模型名称，直接拷贝列表中的模型 ID 到配置文件。

**配置目录**：

- win

```
C:\Users\<你的用户名>\.config\opencode\agents
```

- macOS

```
~/.config/opencode/agents

进入该目录编辑对应的 Agent 配置文件，即可更改使用的模型。
```
---

## 六、TUI 命令速查表

```
/connect        添加 Provider
/models         查看模型列表
/new            新建会话
/sessions       会话列表/切换
/compact        压缩会话
/undo           撤销
/redo           重做
/details        工具执行详情
/thinking       显示/隐藏思考过程
/editor         外部编辑器
/export         导出 Markdown
/themes         换主题
/share          分享会话
/help           帮助
/exit           退出
```

---

## 七、总结

今天的核心收获：

1. **Tab 键**是模式切换的核心，熟悉 Plan/Build/Superpowers 三种模式
2. **Superpowers** 是目前最结构化的开发工作流，适合复杂功能开发
3. **Ctrl+X 快捷键**覆盖了 90% 的日常操作，熟练后效率大幅提升
4. **子 Agent 导航**（↑↓←→）是观察 Superpowers 工作过程的关键
5. **会话管理**（session list / --continue）确保不丢工作进度

OpenCode 在 Windows 下的体验已经比较成熟，配合微信小程序项目使用，是很不错的 AI 辅助开发工具。

---

*文档生成时间：2026-05-03*
*工具版本：opencode v1.14.33*