---
layout: post
title: "Claude Code 完全指南：从安装到多Agent协作"
subtitle: "系统性地掌握 Claude Code CLI 的安装、Skills、Agent、Agent Teams 与 MCP"
date: 2026-02-16
author: Jackson
header-img: "img/2027/timg.jpg"
tags:
  - Claude Code
  - AI
  - 开发者工具
  - MCP
---

> Claude Code 是 Anthropic 推出的 AI 编程助手，它不仅是一个增强版的代码补全工具，而是一个真正的智能体开发系统。本文将带你系统性地掌握 Claude Code 的核心功能。

## 什么是 Claude Code？

Claude Code 是一个**智能体系统（Agentic System）**，而非简单的聊天界面。它通过命令行界面直接在终端中运行，能够：

- 读取、编辑、创建文件
- 执行终端命令
- 使用工具与外部系统交互
- 协调多个子 Agent 并行工作
- 通过 MCP 协议连接外部服务

## 安装与配置

### 系统要求

| 要求 | 详情 |
|------|------|
| 操作系统 | macOS 13+, Ubuntu 20.04+, Windows 10+ |
| 内存 | 4GB 以上 |
| 网络 | 需要互联网连接（API 调用） |

### 安装方法

**macOS / Linux / WSL：**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell：**

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD：**

```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

**使用 Homebrew（macOS）：**

```bash
brew install anthropic/claude-code/claude-code
```

### 验证安装

```bash
claude --version
```

### 认证配置

安装完成后，运行 `claude` 并通过 OAuth 完成认证。你需要：

1. 访问 Claude Console
2. 完成 OAuth 流程
3. 设置计费方式（Pro/Max 订阅或 Console 计费）

---

## 配置系统

Claude Code 使用分层配置系统：

```json
// 优先级从高到低
// 1. 项目本地: .claude/settings.json
// 2. 用户级: ~/.claude/settings.json
// 3. 系统级: /etc/claude-code/settings.json
```

### 常用配置项

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": ["Read", "Glob", "Grep", "Bash"],
    "deny": ["Bash", "rm -rf /"]
  },
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 环境变量

```bash
# 模型选择
ANTHROPIC_MODEL=claude-opus-4-6
ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-5-20250929

# 子 Agent 模型
CLAUDE_CODE_SUBAGENT_MODEL=sonnet

# 扩展思考
MAX_THINKING_TOKENS=10000

# Agent Teams 实验功能
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# MCP 配置
MCP_TIMEOUT=5000
```

---

## 核心交互模式

### 1. 交互模式（默认）

```bash
claude
claude "解释这个项目"
```

### 2. 打印模式（单次查询）

```bash
claude -p "这个函数是做什么的？"
```

### 3. 任务模式

```bash
claude --agent plan  # 使用规划 Agent
claude --agent explore  # 使用探索 Agent
```

### 4. Agent Teams 模式

```bash
# 需要先启用 Agent Teams
claude --teammate-mode auto
```

---

## Skills 系统

Skills 是 Claude Code 最强大的扩展机制之一。它们是**自主运行的背景助手**，能够根据触发条件自动激活。

### Skills 工作原理

每个 Skill 包含：
- **YAML 前置配置**：定义触发条件和行为
- **系统提示**：Skill 的专业领域知识
- **工具集**：特定的可用工具

### 触发类型

| 触发类型 | 说明 |
|----------|------|
| `file_save` | 文件保存时触发 |
| `Bash` | 执行命令时触发 |
| `Read` | 读取文件时触发 |
| `Glob` | 文件搜索时触发 |
| `Grep` | 内容搜索时触发 |

### 安装 Skills

```bash
# 方式1: 使用 claude mcp add
claude mcp add skill-name -- npx -y @package/name

# 方式2: 手动复制
cp -r /path/to/skill ~/.claude/skills/
```

### 常用 Skills 推荐

| Skill | 功能 |
|-------|------|
| `/commit` | 智能 Git 提交 |
| `/review` | 深度代码审查 |
| `/test` | 生成测试用例 |
| `/scaffold` | 脚手架生成 |

### 创建自定义 Skill

```yaml
# ~/.claude/skills/my-skill/skill.md
---
name: my-skill
description: 我的自定义技能
trigger: file_save
---

# 系统提示
你是一个专业的代码审查专家...
```

---

## Subagents 子代理系统

Subagents 是 Claude Code 的任务委派机制，用于处理复杂的多步骤任务。

### 内置 Subagents

| Agent | 用途 |
|-------|------|
| `general` | 通用任务 |
| `explore` | 快速探索代码库 |
| `plan` | 任务规划 |
| `review` | 代码审查 |

### 使用 Subagents

```bash
claude --agent explore  # 启动探索 Agent
```

### 自定义 Subagents

在 `.claude/agents/` 目录创建：

```yaml
# ~/.claude/agents/security-reviewer.md
---
name: security-reviewer
description: 安全代码审查专家
model: sonnet
tools:
  - Read
  - Glob
  - Grep
  - Bash
---

你是一个专业的安全工程师...
```

### 并行 Subagents（7-Agent 模式）

在 `CLAUDE.md` 中配置：

```markdown
## Parallel Feature Implementation Workflow
When implementing features, spawn 7 parallel Task agents:
1. **Component**: Create main component file
2. **Styles**: Create component styles/CSS
3. **Tests**: Write unit tests
4. **Types**: Define TypeScript types
5. **API**: Create API endpoints
6. **Docs**: Write documentation
7. **E2E**: Create E2E tests
```

---

## Agent Teams 多智能体协作

**Agent Teams** 是 Claude Code 最强大的功能——多个 Claude 实例并行工作，模拟真实的工程团队协作。

### 启用 Agent Teams

```json
// ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### Agent Teams 架构

```
┌─────────────────────────────────────────────┐
│              Team Lead                       │
│  - 协调工作                                  │
│  - 分配任务                                  │
│  - 汇总结果                                  │
└──────────────┬──────────────────────────────┘
               │
    ┌──────────┼──────────┬────────────┐
    ▼          ▼          ▼            ▼
┌───────┐ ┌───────┐ ┌───────┐   ┌───────┐
│ Teammate│ │Teammate│ │Teammate│   │Teammate│
│ API层   │ │前端组件 │ │测试    │   │文档    │
│ 独立上下文│ │独立上下文│ │独立上下文│  │独立上下文│
└───────┘ └───────┘ └───────┘   └───────┘
```

### 使用示例

```bash
# 创建一个团队来重构支付模块
Create an agent team to refactor the payment module. 
Spawn three teammates: one for API layer, one for database migrations, 
one for test coverage. Coordinate through shared task list.
```

### 团队模式

| 模式 | 说明 |
|------|------|
| `auto` | 自动选择最佳模式 |
| `in-process` | 进程内运行 |
| `tmux` | 使用 tmux 会话 |

```bash
claude --teammate-mode tmux
```

### 适用场景

- 大型功能实现
- 全面代码审查
- 多维度 QA 测试
- 并行研究任务

---

## MCP（Model Context Protocol）

MCP 是一个开放标准，用于将 AI 智能体连接到外部工具和数据源。

### MCP 架构

```
┌─────────────┐     MCP      ┌─────────────┐
│ Claude Code │ ◄──────────► │ MCP Server  │
│  (Client)   │              │  (Server)   │
└─────────────┘              └─────────────┘
        │                            │
        │                            ▼
        │                   ┌─────────────┐
        └──────────────────►│ External    │
                            │ Tools/APIs  │
                            └─────────────┘
```

### MCP Server 类型

| 类型 | 说明 |
|------|------|
| Local | 本地运行的进程 |
| Remote | 远程托管的服务 |
| Hybrid | 本地进程连接云服务 |

### 常用 MCP Servers

```bash
# GitHub 集成
claude mcp add github -- npx -y @modelcontextprotocol/server-github

# 搜索历史对话
claude mcp add claude-historian-mcp -- npx claude-historian-mcp

# Playwright 浏览器自动化
claude mcp add playwright -- npx -y @modelcontextprotocol/server-playwright

# Figma 设计集成
claude mcp add figma -- npx -y @modelcontextprotocol/server-figma

# PostgreSQL 数据库
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
```

### 配置 MCP

方式1: 命令行

```bash
claude mcp add server-name -- npx -y @package/name
```

方式2: 配置文件

```json
// ~/.claude/settings.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    }
  }
}
```

### 管理 MCP

```bash
# 列出已安装的 MCP
claude mcp list

# 移除 MCP
claude mcp remove server-name
```

---

## 实践工作流

### 日常开发流程

```bash
# 1. 进入项目
cd my-project
claude

# 2. 使用 Agent Teams 进行大功能开发
Create a team to build the new dashboard feature.
One teammate on API, one on frontend, one on tests.

# 3. 完成后使用 Skill 进行提交
/gitcommit
```

### CLAUDE.md 最佳实践

在项目根目录创建 `CLAUDE.md`：

```markdown
# Project Context

## Language & Framework
- TypeScript
- Next.js 14
- pnpm

## Key Paths
- /app - 应用代码
- /components - 组件
- /lib - 工具函数

## Commands
- pnpm dev - 启动开发服务器
- pnpm build - 构建生产版本
- pnpm test - 运行测试

## Code Style
- 使用 ESLint + Prettier
- 函数式组件 + TypeScript
- CSS Modules

## Parallel Implementation
When implementing features, spawn 7 parallel Task agents...
```

---

## 成本优化

### 使用 ccusage 监控

```bash
npm install -g ccusage
ccusage
```

### 最佳实践

1. **使用 Sonnet 处理大多数任务**，Opus 留给复杂推理
2. **及时使用 `/resume` 或 Ctrl+C** 终止无用思考
3. **使用 `Read` 替代 `Grep` 进行精确查找**
4. **禁用不需要的 MCP Servers**

---

## 常见问题

### Q: Claude Code 与 VS Code 扩展有什么区别？
A: Claude Code 是独立的 CLI 工具，可完全控制终端操作；VS Code 扩展是 IDE 内的辅助工具。

### Q: Agent Teams 与 Subagents 有什么区别？
A: Subagents 在单一会话内运行，共享上下文；Agent Teams 是多个独立会话，可直接互相通信。

### Q: MCP 安全吗？
A: MCP 只在配置的权限范围内操作。你可以在 `settings.json` 中精细控制每个 MCP 的权限。

---

## 总结

Claude Code 是一个强大的智能体开发系统，核心包括：

1. **配置系统**：分层配置控制行为
2. **权限系统**：精细控制操作范围
3. **Hook 系统**：确定性自动化
4. **MCP 协议**：扩展外部能力
5. **Subagent 系统**：处理复杂多步骤任务
6. **Agent Teams**：多智能体并行协作

掌握这些核心系统，你将获得一个真正强大的 AI 开发伙伴。

---

*本文基于 Claude Code v2.1+ 版本编写。*
