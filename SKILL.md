---
name: claude-code-cli
description: |
  调用 Claude Code CLI 进行复杂代码开发任务。使用场景包括：
  (1) 完整功能开发 - 从需求到实现的全流程
  (2) 代码审查 - 安全、性能、最佳实践审查
  (3) Bug 追踪与修复 - 堆栈分析到修复验证
  (4) 大规模重构 - 多文件并行重构
  (5) 测试生成 - 单元测试、集成测试、E2E
  (6) PR 创建与管理 - commit、PR、代码合并
  (7) 代码库探索 - 理解新代码库、查找相关代码
  (8) 使用子代理和内置技能 - /simplify, /batch, /debug 等
  适用于 OpenClaw 通过 PTY 调用 Claude Code 的场景。
---

# Claude Code CLI 深度集成

本 skill 提供 Claude Code 完整功能的集成，用于处理复杂代码开发任务。

## 核心调用模式

### 1. 基础调用 (单次查询)

```bash
claude -p "任务描述" --dangerously-skip-permissions [--dir /path/to/project]
```

### 2. 交互式会话

```bash
# 启动交互式会话
claude

# 继续最近会话
claude -c

# 继续指定会话
claude -r "session-name" "继续任务"
```

### 3. Plan Mode (安全分析模式)

只读分析，不执行任何修改：

```bash
claude --permission-mode plan -p "分析认证系统并提出改进建议"
```

适合：
- 代码库研究
- 规划复杂重构
- 安全审查
- 架构分析

## 高级功能

### 子代理 (Sub-agents)

Claude Code 内置专业子代理：

| 子代理 | 用途 | 工具 |
|--------|------|------|
| **Explore** | 快速搜索和分析代码库 | 只读 (Read, Glob, Grep) |
| **Plan** | 研究代码库用于规划 | 只读 |
| **General-purpose** | 复杂多步骤任务 | 全部工具 |

自动调用示例：
```bash
claude -p "探索这个代码库的架构" --dangerously-skip-permissions
```

### 内置 Skills

Claude Code 提供强大的内置 skills：

| Skill | 功能 | 示例 |
|-------|------|------|
| `/simplify` | 审查最近修改，修复代码质量、可重用性、效率问题 | `/simplify` 或 `/simplify focus on memory efficiency` |
| `/batch` | 大规模并行重构 (5-30 个独立任务) | `/batch migrate src/ from Solid to React` |
| `/debug` | 排查当前会话问题 | `/debug` |
| `/loop` | 周期性运行任务 | `/loop 5m check if deploy finished` |
| `/claude-api` | 加载 API 参考文档 | 自动加载 |

### 工作流

#### 理解新代码库

```bash
# 快速概览
claude -p "give me an overview of this codebase" --dangerously-skip-permissions --dir /path

# 架构分析
claude -p "explain the main architecture patterns" --dangerously-skip-permissions --dir /path

# 查找相关代码
claude -p "find files related to user authentication" --dangerously-skip-permissions --dir /path

# 追踪执行流程
claude -p "trace the login process from front-end to database" --dangerously-skip-permissions --dir /path
```

#### Bug 修复

```bash
# 提供错误信息
claude -p "I'm seeing error: Cannot read property 'foo' of undefined in user.ts" --dangerously-skip-permissions --dir /path

# 建议修复方案
claude -p "suggest ways to fix the null pointer issue in auth.ts" --dangerously-skip-permissions --dir /path

# 应用修复
claude -p "apply the null check fix to auth.ts and run tests" --dangerously-skip-permissions --dir /path
```

#### 代码重构

```bash
# 识别需要重构的代码
claude -p "find deprecated API usage in the codebase" --dangerously-skip-permissions --dir /path

# 获取重构建议
claude -p "suggest how to refactor utils.js to modern JavaScript" --dangerously-skip-permissions --dir /path

# 执行重构
claude -p "refactor utils.js to use ES2024 features" --dangerously-skip-permissions --dir /path
```

#### 测试生成

```bash
# 查找未测试代码
claude -p "find functions not covered by tests" --dangerously-skip-permissions --dir /path

# 生成测试
claude -p "add unit tests for the user service" --dangerously-skip-permissions --dir /path

# 边界条件测试
claude -p "add test cases for edge conditions in payment.ts" --dangerously-skip-permissions --dir /path

# 运行并验证
claude -p "run tests and fix any failures" --dangerously-skip-permissions --dir /path
```

#### Git 操作

```bash
# 创建 commit
claude -p "create a commit with meaningful message" --dangerously-skip-permissions

# 创建分支并 commit
claude -p "create feature branch and commit changes" --dangerously-skip-permissions

# 打开 PR (需要 gh CLI)
claude -p "create a pull request for the changes" --dangerously-skip-permissions
```

## 复杂任务模式

### 1. 多步骤任务

```bash
claude -p "build a user authentication module with login, register, and password reset. Include JWT tokens, bcrypt hashing, and input validation. Write tests for all endpoints." --dangerously-skip-permissions --dir /path
```

### 2. 大规模并行任务 (/batch)

```bash
# 在 Claude Code 交互模式中运行
/batch migrate src/ from JavaScript to TypeScript
```

### 3. 持续会话 + 任务

```bash
# 启动长时间会话
claude --dir /path

# 然后在会话中逐步完成任务
# 1. 先探索代码库
# 2. 创建详细计划
# 3. 逐步实现
# 4. 编写测试
# 5. 创建 PR
```

## 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-p` | 单次查询模式 | `claude -p "task"` |
| `-c` | 继续最近会话 | `claude -c` |
| `-r` | 恢复指定会话 | `claude -r session-name` |
| `--dir` | 指定工作目录 | `--dir C:/project` |
| `--permission-mode` | 权限模式 | `--permission-mode plan` |
| `--dangerously-skip-permissions` | 跳过权限确认 | (谨慎使用) |
| `--model` | 指定模型 | `--model sonnet` |
| `--add-dir` | 添加额外目录 | `--add-dir ../shared-lib` |
| `--allowedTools` | 允许的工具 | `--allowedTools "Read" "Bash(git *)"` |

## 模型选择

| 模型 | 用途 | 速度 |
|------|------|------|
| `haiku` | 快速探索、简单任务 | 最快 |
| `sonnet` | 平衡 (推荐) | 中等 |
| `opus` | 复杂推理、重构 | 较慢 |

## OpenClaw 集成示例

### 完整功能开发流程

```bash
# 1. 探索代码库
claude -p "探索项目结构和技术栈" --dangerously-skip-permissions --dir C:/project

# 2. 规划功能
claude -p "创建用户管理模块的详细实现计划" --dangerously-skip-permissions --dir C:/project

# 3. 实现功能 (可在交互会话中)
claude --dir C:/project
# 在会话中: "实现用户注册功能，包括验证、加密、数据库存储"
```

### 代码审查流程

```bash
# 1. 代码质量审查
claude -p "review recent changes for code quality" --dangerously-skip-permissions --dir C:/project

# 2. 安全审查
claude -p "review for security vulnerabilities" --dangerously-skip-permissions --dir C:/project

# 3. 性能审查
claude -p "review for performance issues" --dangerously-skip-permissions --dir C:/project
```

### 使用 /simplify 清理代码

```bash
# 在交互模式中使用
claude --dir C:/project
# 输入: /simplify
# 或: /simplify focus on memory efficiency
```

## 安装和认证

### 检查安装状态

```powershell
claude auth status
```

### 安装 Claude Code (Windows PowerShell)

```powershell
irm https://claude.ai/install.ps1 | iex
```

或使用 WinGet:

```powershell
winget install Anthropic.ClaudeCode
```

### 登录

```powershell
claude auth login --email user@example.com
# 支持 SSO
claude auth login --sso
```

---

## 完整 CLI 参数参考

| 参数 | 说明 | 示例 |
|------|------|------|
| `-p, --print` | 非交互式，打印结果后退出 | `claude -p "task"` |
| `-c, --continue` | 继续最近会话 | `claude -c` |
| `-r, --resume` | 恢复指定会话 | `claude -r session-name` |
| `--model` | 指定模型 (sonnet/haiku/opus/MiniMax-M2.5) | `--model sonnet` |
| `--max-turns` | 限制执行轮次 | `--max-turns 3` |
| `--max-budget-usd` | 最大 API 花费 (print 模式) | `--max-budget-usd 5.00` |
| `--dangerously-skip-permissions` | 跳过权限确认 (慎用) | |
| `--output-format` | 输出格式 (text/json/stream-json) | `--output-format json` |
| `--input-format` | 输入格式 | `--input-format stream-json` |
| `--permission-mode` | 权限模式 (plan/auto/medium) | `--permission-mode plan` |
| `--allowedTools` | 允许的工具 (逗号分隔) | `--allowedTools Read,Bash` |
| `--disallowedTools` | 禁止的工具 | `--disallowedTools Edit` |
| `--add-dir` | 添加额外工作目录 | `--add-dir ../lib` |
| `--agent` | 指定子代理 | `--agent my-agent` |
| `--agents` | 动态定义子代理 (JSON) | |
| `--chrome` | 启用 Chrome 集成 | `--chrome` |
| `--from-pr` | 从 GitHub PR 恢复会话 | `--from-pr 123` |
| `--betas` | 启用 Beta 功能 | `--betas interleaved-thinking` |
| `--debug` | 调试模式 | `--debug "api,mcp"` |
| `--fallback-model` | 备用模型 | `--fallback-model haiku` |

---

## 权限模式

Claude Code 有三种权限模式：

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| `plan` | 只读分析，不修改文件 | 代码审查、安全检查 |
| `auto` | 自动执行，无需确认 | 快速开发 (谨慎使用) |
| `medium` | 执行前确认 | 日常开发 (默认) |

### 使用方式

```powershell
# Plan 模式 - 安全分析
claude --permission-mode plan -p "分析认证系统的安全问题"

# 交互式切换：Shift+Tab 循环切换模式
```

---

## MCP 集成 (Model Context Protocol)

MCP 让 Claude Code 连接外部工具和服务。

### 添加 MCP Server

```powershell
# stdio 模式
claude mcp add server-name --transport stdio -- env VAR=value -- npx -y mcp-server

# HTTP 模式
claude mcp add server-name --transport http https://mcp-server.example.com

# SSE 模式
claude mcp add server-name --transport sse https://mcp-server.example.com
```

### 列出 MCP Servers

```powershell
claude mcp list
```

### 常用 MCP Servers

- **GitHub**: 代码库管理、PR、Issue
- **Filesystem**: 文件系统操作
- **Database**: 数据库查询
- **Slack/Discord**: 消息通知
- **Jira**: 任务管理

---

## 在 OpenClaw 中的最佳实践

### 1. 简单任务用 Print 模式

```powershell
# OpenClaw 调用示例 (使用 exec + pty)
claude -p "审查 src/auth.js 的安全性" --permission-mode plan
```

### 2. 复杂任务用交互模式

```powershell
# 需要多轮交互时
claude "重构整个认证模块，使用 OAuth2"
```

### 3. 敏感操作用 Plan 模式

```powershell
# 代码审查、安全分析
claude --permission-mode plan -p "分析这个 PR 的安全风险"
```

### 4. 限制资源使用

```powershell
# 限制花费
claude -p --max-budget-usd 2.00 "修复这个 bug"

# 限制轮次
claude -p --max-turns 10 "重构这个函数"
```

### 5. 继续之前的工作

```powershell
# 继续最近会话
claude -c

# 继续指定会话
claude -r session-name "继续完成这个任务"
```

---

## 工具调用流程 (OpenClaw)

1. **检查 Claude Code 是否可用**
   ```powershell
   claude auth status
   ```

2. **选择调用模式**
   - 简单任务 → `--print` 模式
   - 复杂任务 → 交互式模式 (需要 PTY)
   - 继续工作 → `--continue` / `--resume`

3. **选择权限模式**
   - 只读分析 → `--permission-mode plan`
   - 日常开发 → `--permission-mode medium` (默认)
   - 自动化脚本 → `--permission-mode auto` (谨慎)

4. **执行并获取结果**

---

## 注意事项

1. **PTY 必需**: Claude Code 交互式模式需要 TTY，OpenClaw 必须使用 `pty: true`
2. **需要登录**: Claude Code 需要 Anthropic 账户
3. **认证**: 首次使用需 `claude auth login`
4. **权限**: 使用 `--dangerously-skip-permissions` 跳过每次确认 (谨慎)
5. **会话**: `-p` 是单次调用；需持续会话时使用 `claude -c` 或交互模式
6. **目录**: 明确指定 `--dir` 确保在正确项目中操作
7. **复杂任务**: 推荐启动交互式会话，分步骤完成
8. **Print 模式限制**: 能力有限，复杂任务用交互式模式
9. **资源限制**: 生产环境建议设置 `--max-budget-usd`
10. **会话管理**: 使用 `-c` / `-r` 继续之前的工作

## 第三方模型提供商配置

Claude Code 支持配置第三方兼容的模型 API 服务商，如阿里云百炼。

### 配置文件位置

```
%USERPROFILE%\.claude\settings.json
```

### 阿里云百炼配置示例

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your-api-key-here",
    "ANTHROPIC_BASE_URL": "https://coding.dashscope.aliyuncs.com/apps/anthropic",
    "API_TIMEOUT_MS": "3000000"
  }
}
```

### 配置说明

| 环境变量 | 说明 | 示例值 |
|----------|------|--------|
| `ANTHROPIC_AUTH_TOKEN` | API Key (百炼控制台获取) | `sk-sp-xxxxxx` |
| `ANTHROPIC_BASE_URL` | API 端点地址 | `https://coding.dashscope.aliyuncs.com/apps/anthropic` |
| `API_TIMEOUT_MS` | 请求超时时间 (毫秒) | `3000000` |

### 注意事项

1. **Base URL 格式**: 结尾**不要**包含 `/v1`，Claude Code 会自动添加
   - ❌ 错误: `https://xxx.com/apps/anthropic/v1`
   - ✅ 正确: `https://xxx.com/apps/anthropic`

2. **API Key 获取**: 登录阿里云百炼控制台 → 模型服务 → API-KEY

3. **可用模型**: 通过 `--model` 参数指定，如 `MiniMax-M2.5`

### 测试配置

```powershell
# 使用 PowerShell 测试 API
$body = @{
    model = "MiniMax-M2.5"
    max_tokens = 20
    messages = @(
        @{role = "user"; content = "hi"}
    )
} | ConvertTo-Json -Depth 3

Invoke-RestMethod -Uri "https://coding.dashscope.aliyuncs.com/apps/anthropic/v1/messages" `
    -Method Post `
    -Headers @{
        "Authorization" = "Bearer YOUR_API_KEY"
        "Content-Type" = "application/json"
    } `
    -Body $body
```

### 指定模型运行

```bash
claude -p "你的任务" --model MiniMax-M2.5
```

## Spec-Kit 集成 (规范化开发流程)

Spec-Kit 是一套规范化的开发流程命令，用于将需求完整转换为可交付代码。

### 指挥命令序列

| 步骤 | 命令 | 产出 |
|------|------|------|
| **1** | `/speckit.specify [需求描述]` | spec.md（用户故事、验收标准） |
| **2** | `/speckit.plan [分支名]` | plan.md、data-model.md |
| **3** | `/speckit.tasks [分支名]` | tasks.md（任务清单） |
| **4** | `/speckit.implement [分支名]` | 源代码实现 |
| **5** | `/speckit.checklist [分支名] [清单名]` | 验收检查 |
| **6** | `/speckit.analyze [分支名]` | 一致性分析 |

### 指挥话术示例

```bash
# 1. 规格定义 - 输入完整需求描述
/speckit.specify 创建拟物化风格的Todo List应用，使用皮革笔记本、纸质便签、金属订书钉效果...

# 2. 实施计划 - 指定分支名
/speckit.plan 001-skeletal-todo

# 3. 任务分解 - 指定分支名
/speckit.tasks 001-skeletal-todo

# 4. 执行实现 - Claude Code 按任务清单开发
/speckit.implement 001-skeletal-todo

# 5. 检查验收 - 验证需求完成度
/speckit.checklist 001-skeletal-todo requirements

# 6. 一致性分析 - 确保设计与实现一致
/speckit.analyze 001-skeletal-todo
```

### Spec-Kit 产出文档结构

```
项目目录/
├── .claude/commands/
│   └── speckit.*.md          # 9个 spec-kit 命令定义
├── .specify/
│   ├── memory/
│   │   ├── spec.md           # 功能规格说明
│   │   ├── plan.md           # 实现计划
│   │   ├── tasks.md          # 任务清单
│   │   └── constitution.md   # 项目宪法
│   ├── scripts/powershell/   # 自动化脚本
│   └── templates/            # 文档模板
└── specs/
    └── [分支名]/
        ├── spec.md           # 完整特性规格
        ├── plan.md           # 实施计划
        ├── data-model.md     # 数据模型
        ├── research.md       # 技术调研
        ├── quickstart.md     # 快速开始
        ├── checklists/        # 验收清单
        └── contracts/        # 契约文档
```

### 闭环验证要点

| 闭环要素 | 验证方式 |
|---------|----------|
| 需求 → 规格 | spec.md 包含用户故事和验收标准 |
| 规格 → 计划 | plan.md 包含技术栈和实现阶段 |
| 计划 → 任务 | tasks.md 每个任务有明确 ID 和文件路径 |
| 任务 → 代码 | 源代码文件存在且功能完整 |
| 代码 → 验收 | checklist 验证通过 |
| 验收 → 闭环 | Success Criteria 全部满足 |

### 协作规则（适用于 OpenClaw）

1. **只记录，不写代码**: 不直接编写代码，只记录 Claude Code 工作过程
2. **任务汇报**: 每次任务完成后必须汇报执行结果
3. **超时处理**: 用户 10 分钟没回答 → 按方案处理（不破坏外部环境）
4. **项目目录**: 新项目建在 `F:\ai-workspace`

---

## 故障排除

```bash
# 查看认证状态
claude auth status

# 更新 Claude Code
claude update

# 调试模式
claude --debug

# 查看可用子代理
claude agents
```