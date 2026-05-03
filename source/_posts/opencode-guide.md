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
| 可执行文件 | ~\AppData\Roaming\QClaw\npm-global\opencode.ps1 |
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

### 3.1 添加 AI Provider（API Key）

```powershell
# 方式一：命令行引导
opencode auth login

# 方式二：直接编辑凭证文件
# 路径：~/.local/share/opencode/auth.json
```

支持的 Provider：OpenAI、Anthropic、Groq、OpenRouter、Azure、Gemini、Bedrock 等。

### 3.2 查看可用模型

```powershell
# 列出所有可用模型
opencode models

# 只看某个 provider 的模型
opencode models anthropic

# 刷新模型列表
opencode models --refresh

# 详细模式（含价格）
opencode models --verbose
```

### 3.3 配置 Agent 模型（opencode.json）

在项目根目录或 `~/.config/opencode/opencode.json` 配置：

```json
{
  "provider": "qwen",
  "model": "qwen3.6-plus",
  "options": {
    "temperature": 0.7
  }
}
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

## 五、Superpowers 安装与使用

### 5.1 什么是 Superpowers

Superpowers 是 OpenCode 的一个多步 Agent 工作流，把开发过程拆成标准步骤：

```
头脑风暴 → 写规格文档 → 审计规格 → 写实现计划 → 执行编码
```

每个步骤需要用户确认后再继续，比直接让 AI 改代码更可控。

### 5.2 安装步骤

```powershell
# 搜索 Superpowers
opencode agent install opencode-superpowers

# 安装完成后查看
opencode agent list
```

成功安装后会看到 5 个 Agent：

| Agent | 角色 |
|-------|------|
| `superpowers` | 主协调器（orchestrator） |
| `superpowers-spec-writer` | 规格编写 |
| `superpowers-spec-auditor` | 规格审计 |
| `superpowers-plan-writer` | 实现计划编写 |
| `superpowers-implementer` | 执行编码 |

以及 6 个 Skills：

- brainstorming - 头脑风暴
- writing-plans - 计划编写
- executing-plans - 计划执行
- verification-before-completion - 完成前验证
- subagent-driven-development - 子代理驱动开发
- using-superpowers - 工作流启动引导

### 5.3 使用方式

```powershell
# 启动 Superpowers 工作流
opencode --agent superpowers
```

进入 TUI 后，切换到 **Superpowers 模式**（按 Tab），然后输入任务描述，例如：

```
帮我把水卡页面改造成角色自适应的管理页面
```

Superpowers 会自动：
1. 先进行头脑风暴（brainstorming）
2. 输出规格文档让你确认
3. 审计规格找问题
4. 生成实现计划
5. 逐任务执行编码

### 5.4 各子 Agent 模型配置

默认所有子 Agent 使用免费模型 `opencode/big-pickle`。如需切换，可在 Agent 文件中修改：

路径：`~/.config/opencode/agents/`

| 文件 | 当前模型 |
|------|---------|
| `superpowers.md` | `qwen3.6-plus` |
| `superpowers-spec-writer.md` | `qwen3.6-plus` |
| `superpowers-spec-auditor.md` | `qwen3.6-plus` |
| `superpowers-plan-writer.md` | `qwen3.6-plus` |
| `superpowers-implementer.md` | `qwen3.6-plus` |

---

## 六、项目实战：改进微信小程序

### 6.1 项目信息

- **项目名称**：智能取水系统 SaaS
- **项目路径**：`D:\WeChatProjects\ysj-project`
- **技术栈**：微信小程序前端 + Java Spring Boot 后端 + 微信云开发
- **角色**：普通用户、租户管理员、超级管理员（三级权限）

### 6.2 OpenCode 实战计划（10 步）

以下是完整的学习实战路线：

1. **首页交互优化** - 优化取水交互流程和页面展示
2. **水卡管理页面** - 完善水卡余额、充值、消费记录
3. **消费记录页面** - 添加筛选、导出、数据可视化
4. **设备管理** - 设备状态监控、故障告警
5. **用户反馈系统** - 报修、投诉、建议提交
6. **租户管理后台** - 租户管理员的完整功能
7. **超级管理后台** - 全局统计、数据管理
8. **云函数开发** - 云端业务逻辑
9. **CI/CD 自动化** - 自动化构建与发布
10. **性能优化** - 包体积、加载速度、内存占用

### 6.3 具体操作示例

**启动 OpenCode 并选择 Agent：**

```powershell
cd D:\WeChatProjects\ysj-project
opencode --agent superpowers
```

**按 Tab 切换到 Superpowers 模式，然后输入：**

```
优化首页的取水交互流程，让用户能更快看到设备状态和取水进度
```

Superpowers 会自动进入规格 → 审计 → 计划的完整流程。

---

## 七、踩坑记录

### 坑1：PowerShell 不支持 && 链式命令

```powershell
# 报错：语法错误
tree /f && cd backend && dir

# 解决：用分号隔开
tree /f; cd backend; dir
```

### 坑2：文件树命令文件名含特殊字符报错

项目中有文件名 `[Help`（疑似 Windows 帮助文件残留），导致 `tree` 命令退出码 1。用普通 `dir` 代替即可。

### 坑3：auth.json 格式问题

配置 Provider API Key 时，写入 `auth.json` 格式不对会导致 `opencode auth list` 显示 0 credentials。正确格式应参考官方文档。

### 坑4：会话恢复

不小心退出 OpenCode 后，可用以下方式回到上一个会话：

```powershell
# 继续最近一次会话
opencode -c

# 指定会话 ID
opencode --session <session_id>

# 查看所有会话
opencode session list
```

---

## 八、TUI 命令速查表

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

## 九、总结

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
*工作目录：D:\WeChatProjects\ysj-project*