---
layout: post
title: "OpenCode 完全指南：开源 AI 编程 Agent"
subtitle: "从安装到深入使用，全面掌握 OpenCode 的 Skills、Agents、MCP 等高级功能"
date: 2026-02-15
author: Jackson
header-img: "img/2017/timg.jpg"
tags:
  - OpenCode
  - AI
  - 开发者工具
  - MCP
  - Agent
---

> OpenCode 是由 Anomaly 公司开发的开源 AI 编程 Agent，在 GitHub 上已获得超过 100,000 颗星标，每月被超过 250 万开发者使用。本文将带你全面掌握 OpenCode 的核心功能。

## 什么是 OpenCode？

OpenCode 是一个**开源 AI 编程 Agent**，提供终端界面、桌面应用和 IDE 插件三种使用方式。它能够：

- **LSP 自动加载**：自动为 LLM 加载正确的语言服务器
- **多会话支持**：在同一项目上启动多个 Agent 并行工作
- **分享链接**：分享会话链接以便参考或调试
- **支持任意模型**：通过 75+ LLM 提供商（包括本地模型）
- **隐私优先**：不存储任何代码或上下文数据

## 安装与配置

### 安装方法

**通用安装（推荐）：**

```bash
curl -fsSL https://opencode.ai/install | bash
```

**使用 npm：**

```bash
npm install -g opencode-ai
```

**使用 Bun：**

```bash
bun install -g opencode-ai
```

**使用 Homebrew（macOS/Linux）：**

```bash
brew install anomalyco/tap/opencode
```

**Windows（推荐使用 WSL）：**

```bash
# 使用 Chocolatey
choco install opencode

# 使用 Scoop
scoop install opencode
```

### 配置 LLM 提供商

1. 运行 `/connect` 命令，选择 OpenCode 并访问 [opencode.ai/auth](https://opencode.ai/auth)
2. 登录后添加计费方式，获取 API 密钥
3. 在 TUI 中粘贴 API 密钥

## 核心功能与使用技巧

### TUI 基础操作

OpenCode 提供交互式终端界面，直接在当前目录运行：

```bash
opencode
```

或指定项目目录：

```bash
opencode /path/to/project
```

### 文件引用

使用 `@` 符号引用项目中的文件：

```
How is authentication handled in @packages/functions/src/api/index.ts?
```

### 执行 Shell 命令

使用 `!` 前缀执行终端命令：

```
!ls -la
```

### 常用命令

| 命令 | 功能 | 快捷键 |
|------|------|--------|
| `/init` | 创建/更新 AGENTS.md | `ctrl+x i` |
| `/new` | 开始新会话 | `ctrl+x n` |
| `/undo` | 撤销上一步操作 | `ctrl+x u` |
| `/redo` | 重做已撤销的操作 | `ctrl+x r` |
| `/share` | 分享当前会话 | `ctrl+x s` |
| `/compact` | 压缩会话上下文 | `ctrl+x c` |
| `/models` | 列出可用模型 | `ctrl+x m` |
| `/themes` | 切换主题 | `ctrl+x t` |

### 模式切换

使用 **Tab** 键在 Build 和 Plan 模式间切换：

- **Build 模式**：默认模式，所有工具可用，进行实际开发
- **Plan 模式**：只读模式，用于分析代码和规划方案，不执行修改

## Agent 系统

OpenCode 内置两种类型的 Agent：**主 Agent（Primary）** 和 **子 Agent（Subagent）**。

### 内置 Agent

#### 1. Build Agent
- **类型**：主 Agent
- **功能**：默认开发 Agent，拥有所有工具权限

#### 2. Plan Agent
- **类型**：主 Agent
- **功能**：受限的规划分析 Agent
- **权限**：默认阻止文件编辑和 bash 命令执行

#### 3. General Agent
- **类型**：子 Agent
- **功能**：通用研究 Agent，可执行多步骤任务
- **使用方式**：在消息中 `@general`

#### 4. Explore Agent
- **类型**：子 Agent
- **功能**：快速代码探索 Agent，只读
- **使用方式**：在消息中 `@explore`

### 创建自定义 Agent

使用命令创建新 Agent：

```bash
opencode agent create
```

这会启动交互式向导，引导你完成：
1. 选择保存位置（全局/项目级）
2. 描述 Agent 职责
3. 选择可用工具
4. 生成配置文件

### Agent 配置文件示例

```yaml
---
description: Reviews code for best practices and potential issues
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.1
tools:
  write: false
  edit: false
  bash: false
---

You are in code review mode. Focus on:
- Code quality and best practices
- Potential bugs and edge cases
- Performance implications
- Security considerations
```

### Agent 权限配置

在 `opencode.json` 中精细控制权限：

```json
{
  "agent": {
    "build": {
      "permission": {
        "edit": "ask",
        "bash": {
          "*": "ask",
          "git status *": "allow",
          "grep *": "allow"
        }
      }
    }
  }
}
```

权限选项：
- `allow`：允许执行
- `deny`：拒绝执行
- `ask`：执行前请求用户确认

## Skills 系统

Skills 允许你定义可重用的行为指令，OpenCode 会按需加载。

### Skill 文件位置

- 项目级：`.opencode/skills/<name>/SKILL.md`
- 全局级：`~/.config/opencode/skills/<name>/SKILL.md`
- Claude 兼容：`.claude/skills/<name>/SKILL.md`

### Skill 格式

```yaml
---
name: git-release
description: Create consistent releases and changelogs
license: MIT
compatibility: opencode
metadata:
  audience: maintainers
  workflow: github
---

## What I do
- Draft release notes from merged PRs
- Propose a version bump
- Provide a copy-pasteable `gh release create` command

## When to use me
Use this when you are preparing a tagged release.
```

### 权限控制

在 `opencode.json` 中配置 Skill 使用权限：

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "pr-review": "allow",
      "internal-*": "deny",
      "experimental-*": "ask"
    }
  }
}
```

## MCP 服务器

MCP（Model Context Protocol）允许 OpenCode 连接外部工具和服务。

### 本地 MCP 服务器

```json
{
  "mcp": {
    "my-mcp": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-everything"],
      "enabled": true
    }
  }
}
```

### 远程 MCP 服务器

```json
{
  "mcp": {
    "my-remote-mcp": {
      "type": "remote",
      "url": "https://my-mcp-server.com",
      "enabled": true,
      "headers": {
        "Authorization": "Bearer MY_API_KEY"
      }
    }
  }
}
```

### 常用 MCP 服务器示例

#### 1. Sentry（错误追踪）

```json
{
  "mcp": {
    "sentry": {
      "type": "remote",
      "url": "https://mcp.sentry.dev/mcp",
      "oauth": {}
    }
  }
}
```

认证后使用：

```
Show me the latest unresolved issues. use sentry
```

#### 2. Context7（文档搜索）

```json
{
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "{env:CONTEXT7_API_KEY}"
      }
    }
  }
}
```

#### 3. Grep by Vercel（代码搜索）

```json
{
  "mcp": {
    "gh_grep": {
      "type": "remote",
      "url": "https://mcp.grep.app"
    }
  }
}
```

使用方式：

```
What's the right way to handle this? use the gh_grep tool
```

### MCP 认证管理

```bash
# 认证 MCP 服务器
opencode mcp auth <server-name>

# 列出所有 MCP 服务器
opencode mcp list

# 移除认证
opencode mcp logout <server-name>

# 调试连接
opencode mcp debug <server-name>
```

## 实际应用案例

### 案例一：代码重构

```
# 切换到 Plan 模式（Tab 键）
# 输入重构需求
Refactor the authentication logic in @src/auth/index.ts to support JWT tokens.
```

Plan Agent 会分析代码并提出重构方案，确认后切换回 Build 模式执行。

### 案例二：Bug 调试

```
@explore find where the memory leak might be in this React app
```

Explore Agent 会快速搜索代码库，定位潜在问题。

### 案例三：使用 Skill 发布版本

```
# 加载 git-release skill
skill({ name: "git-release" })

Create a new release for version 1.2.0
```

### 案例四：集成 Sentry 监控

1. 配置 Sentry MCP 服务器
2. 在开发时直接查询线上错误：

```
Show me the critical errors in production from last week. use sentry
```

### 案例五：团队协作

1. 使用 `/share` 生成分享链接
2. 将链接发送给团队成员
3. 成员可以直接查看完整对话上下文

### 案例六：创建专业 Agent

创建代码审查 Agent：

```yaml
---
name: code-reviewer
description: Performs security audits and identifies vulnerabilities
mode: subagent
tools:
  write: false
  edit: false
---

You are a security expert. Focus on identifying potential security issues.
Look for:
- Input validation vulnerabilities
- Authentication and authorization flaws
- Data exposure risks
- Dependency vulnerabilities
- Configuration security issues
```

使用方式：

```
@code-reviewer review the authentication module
```

## 高级技巧

### 1. 使用 AGENTS.md 项目配置

创建 `AGENTS.md` 文件帮助 Agent 理解项目结构：

```markdown
# Project: MyApp

## Tech Stack
- Next.js 15 (App Router)
- TypeScript
- Tailwind CSS 4

## Code Patterns
- Use React Server Components by default
- Use server actions for mutations

## Testing
- Vitest for unit tests
- Playwright for E2E tests
```

运行 `/init` 自动生成。

### 2. 利用上下文压缩

当会话过长时，使用 `/compact` 压缩上下文，保留关键信息。

### 3. 外部编辑器集成

设置 `EDITOR` 环境变量，使用熟悉的编辑器撰写提示词：

```bash
export EDITOR="code --wait"
```

然后使用 `/editor` 命令。

### 4. 多会话管理

使用 `/sessions` 在多个会话间切换，继续之前的工作。

### 5. 图像输入

直接将图片拖入终端，OpenCode 可以分析图像内容。

## 总结

OpenCode 是一个功能强大的开源 AI 编程 Agent，其核心优势包括：

1. **完全开源**：100K+ GitHub Stars，社区活跃
2. **灵活配置**：支持任意 LLM 提供商
3. **隐私安全**：不存储代码数据
4. **可扩展**：通过 Skills 和 MCP 无限扩展
5. **多平台**：终端、桌面应用、IDE 插件

无论是个人开发还是团队协作，OpenCode 都能显著提升开发效率。强烈建议将 AGENTS.md 提交到项目仓库，帮助 AI 更好地理解你的代码库。

---

**相关资源：**

- [OpenCode 官网](https://opencode.ai)
- [OpenCode GitHub](https://github.com/anomalyco/opencode)
- [OpenCode 文档](https://opencode.ai/docs)
- [OpenCode Discord](https://opencode.ai/discord)
