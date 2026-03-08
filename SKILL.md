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

## 注意事项

1. **认证**: 首次使用需 `claude auth login`
2. **权限**: 使用 `--dangerously-skip-permissions` 跳过每次确认 (谨慎)
3. **会话**: `-p` 是单次调用；需持续会话时使用 `claude -c` 或交互模式
4. **目录**: 明确指定 `--dir` 确保在正确项目中操作
5. **复杂任务**: 推荐启动交互式会话，分步骤完成

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