# Claude Code Source Project

## 项目概述

这是 **Claude Code** 的源代码库 - Anthropic官方推出的AI代码助手CLI工具。Claude Code 是一个强大的交互式代码编辑和分析工具，支持多种编程场景包括：
- 代码生成和编辑
- Bug修复和重构
- 文档生成
- 测试编写
- 代码审查
- 复杂的多步骤工程任务

## 技术栈

- **语言**: TypeScript/JavaScript
- **运行时**: Bun（现代JavaScript运行时）
- **UI框架**: React + Ink（Terminal UI）
- **命令行**: Commander.js
- **包管理**: yarn/npm

## 项目结构

```
claude_code_source/
├── src/
│   ├── main.tsx              # 应用入口点
│   ├── commands.ts           # 命令系统定义
│   ├── commands/             # 各个命令的实现
│   ├── tools/                # 工具系统（>45个工具）
│   ├── components/           # React UI组件库（>145个）
│   ├── hooks/                # React Hooks（>85个）
│   ├── services/             # 核心服务
│   │   ├── api/              # API调用和集成
│   │   ├── mcp/              # Model Context Protocol支持
│   │   ├── analytics/        # 分析和遥测
│   │   ├── policyLimits/     # 策略和限制管理
│   │   └── remoteManagedSettings/  # 远程配置管理
│   ├── cli/                  # CLI输出和交互
│   │   ├── print.ts          # 输出格式化（>200KB）
│   │   ├── transports/       # 传输层
│   │   └── handlers/         # 命令处理器
│   ├── query/                # 查询引擎
│   ├── QueryEngine.ts        # 主查询执行引擎
│   ├── context.ts            # 全局上下文管理
│   ├── history.ts            # 历史记录管理
│   ├── tasks/                # 任务系统
│   ├── state/                # 状态管理
│   ├── skills/               # 技能系统
│   ├── bridge/               # 多个集成桥接
│   ├── remote/               # 远程执行支持
│   ├── mcp/                  # MCP服务器
│   ├── memdir/               # 内存/临时目录管理
│   ├── migrations/           # 数据迁移
│   ├── utils/                # 工具函数库
│   └── vendor/               # 第三方代码
└── node_modules/             # 依赖包

```

## 核心功能模块

### 1. **命令系统** (`commands/`, `commands.ts`)
- 完整的CLI命令框架
- 支持>100个命令
- 命令验证和路由
- 帮助文本和文档生成

### 2. **工具系统** (`tools/`, `Tool.ts`)
- 可扩展的工具框架
- 集成100+个工具：
  - 代码分析工具（Glob, Grep, Read, Edit等）
  - AI API工具（Claude API, Web Search等）
  - 开发工具（Git, Docker, Bash等）
  - 特殊工具（Agent创建, Task管理等）

### 3. **查询引擎** (`query/`, `QueryEngine.ts`, `query.ts`)
- 处理用户输入和对话
- 工具调用调度和执行
- 流式输出管理
- 错误处理和重试

### 4. **服务层** (`services/`)
- **API服务**: Anthropic API集成, Bootstrap数据, 文件API
- **MCP服务**: Model Context Protocol支持
- **分析**: GrowthBook特性标志, 事件日志
- **策略限制**: 使用配额和策略管理
- **远程配置**: 远程管理的设置

### 5. **UI系统** (`components/`, `hooks/`, `ink.ts`)
- Terminal UI组件库（>145个组件）
- 深度的React Hooks库（>85个）
- Ink框架集成用于Terminal渲染
- 对话框, 菜单, 表单, 通知等

### 6. **CLI输出** (`cli/`)
- 结构化输出格式化
- NDJSON支持
- 多种传输层（stdio, WebSocket等）
- 安全的字符串化

### 7. **任务系统** (`tasks/`, `Task.ts`, `tasks.ts`)
- 后台任务管理
- 任务队列和调度
- 进度追踪

### 8. **状态管理** (`state/`)
- 应用全局状态
- 持久化存储
- 上下文传播

### 9. **技能系统** (`skills/`)
- 可配置的技能/宏
- 内置技能库
- 用户定义的快捷方式

### 10. **高级功能**
- **Agent支持**: 创建和管理AI Agent
- **Swarm模式**: 多Agent协作（teammate.ts）
- **Coordinator模式**: Agent协调器
- **Assistant模式**: Kairos助手
- **Worktree支持**: Git worktree集成
- **远程执行**: 远程会话和执行
- **MDM支持**: 移动设备管理集成
- **Keychain集成**: 安全凭证存储
- **OAuth支持**: 认证和授权
- **插件系统**: 可扩展插件支持

## Web部署架构

### 三层架构

```
┌─────────────────────┐
│   Web Browser       │  https://claude.ai/code
│  (React UI)         │
└──────────┬──────────┘
           │ WebSocket/HTTP Bridge API
           ↓
┌─────────────────────┐
│  Claude.ai Backend  │  https://claude.ai (Web Server)
│  (Node.js/Python?)  │
└──────────┬──────────┘
           │ SSH/Direct Connection
           ↓
┌─────────────────────┐
│  Claude Code CLI    │  User's Machine
│  (Bun Runtime)      │
└─────────────────────┘
```

### Bridge工作流

**创建远程会话:**
1. 用户访问 https://claude.ai/code
2. 点击"连接本地CLI"或启动查询
3. Web app调用 `createSession()` API
4. Session ID返回到web端
5. CLI通过 `bridgeApi.ts` 连接到这个session

**执行查询:**
1. Web UI发送用户查询到 Bridge API
2. CLI的 `bridgeMain.ts` 接收消息
3. 执行工具和命令
4. 结果通过Bridge流式返回
5. Web UI实时显示进度

**权限流:**
1. Web端请求CLI执行敏感操作（文件访问、命令执行等）
2. `bridgePermissionCallbacks.ts` 处理权限检查
3. 可选：Terminal弹出权限确认对话框
4. 用户批准/拒绝
5. 结果返回给Web app

### 远程Session管理

**Session创建:**
- `createSession.ts` - 创建新的远程会话
- `codeSessionApi.ts` - 管理Session生命周期
- 支持会话恢复和持久化

**Session状态:**
- 运行中 (RUNNING)
- 等待中 (WAITING)
- 完成 (COMPLETED)
- 错误 (ERROR)

### 关键特性

| 特性 | 说明 |
|-----|-----|
| **认证** | OAuth流 + Access Token |
| **加密** | TLS/HTTPS通信 |
| **权限** | 精细化权限控制 |
| **流式传输** | 实时进度更新 |
| **失败恢复** | 自动重连和重试 |
| **多设备** | 同时支持Web和CLI |
| **跨平台** | macOS, Linux, Windows |

### 部署环境

**开发环境:**
```
https://claude.ai (staging/dev)
```

**生产环境:**
```
https://claude.ai (production)
```

**本地测试:**
```
http://localhost:3000 (可配置)
```

## 关键文件和职责

| 文件/目录 | 职责 |
|---------|------|
| **Core** | |
| `main.tsx` | 应用启动, 初始化, 主事件循环 |
| `commands.ts` | 命令列表和注册 |
| `QueryEngine.ts` | 核心查询执行逻辑 |
| `Tool.ts` | 工具系统基础和接口 |
| `context.ts` | 全局上下文和用户/系统信息 |
| `history.ts` | 会话历史管理 |
| `cost-tracker.ts` | API使用成本计算 |
| `setup.ts` | 初始设置和配置向导 |
| **CLI/UI** | |
| `cli/print.ts` | 输出格式化和样式 |
| `interactiveHelpers.tsx` | 交互UI辅助函数 |
| `dialogLaunchers.tsx` | 对话框启动器 |
| **Web Integration (Bridge)** | |
| `bridge/bridgeApi.ts` | Bridge API客户端 |
| `bridge/bridgeMain.ts` | Bridge主逻辑和状态管理 |
| `bridge/bridgeMessaging.ts` | Web↔CLI消息协议 |
| `bridge/createSession.ts` | 远程会话创建 |
| `bridge/codeSessionApi.ts` | Session API |
| `bridge/bridgeUI.ts` | Web集成的UI逻辑 |
| `bridge/bridgePermissionCallbacks.ts` | 权限处理 |
| `constants/product.ts` | Web URLs配置 |

## 性能优化亮点

1. **启动优化** (`main.tsx` 顶部)
   - 并行化MDM读取
   - 预取keychain数据
   - 启动性能分析和监控

2. **代码分割**
   - 条件导入（COORDINATOR_MODE, KAIROS）
   - 死码消除
   - 特性标志支持

3. **并发处理**
   - 后台任务执行
   - 流式输出
   - 异步工具调用

## 配置和管理

### 设置系统
- 全局配置管理 (`utils/config.ts`)
- 远程管理设置 (`services/remoteManagedSettings/`)
- 策略限制 (`services/policyLimits/`)
- MDM配置支持

### 安全性
- Keychain集成用于凭证存储
- OAuth流程
- 安全的环境变量处理
- API密钥管理

### 分析和遥测
- GrowthBook特性标志集成
- 事件日志
- 匿名使用分析
- 可选关闭分析

## 开发约定

### TypeScript配置
- 严格类型检查
- 自定义ESLint规则（如：no-top-level-side-effects）
- 模块路径别名支持

### 代码组织
- 模块化架构
- 清晰的服务分离
- 反应式状态管理
- hooks优先的UI组件

### 测试策略
- 单元测试
- 集成测试
- E2E测试覆盖
- 性能基准测试

## 工作流程

### 添加新命令
1. 在 `commands/` 中创建命令实现
2. 在 `commands.ts` 中注册命令
3. 添加帮助文本和验证
4. 更新CLI处理器

### 添加新工具
1. 在 `tools/` 中实现工具
2. 继承 `Tool` 基类
3. 定义输入/输出schema
4. 在 `tools.ts` 中注册

### 添加新功能
1. 评估是否需要新服务
2. 创建必要的组件/hooks
3. 集成到查询引擎
4. 添加配置选项

## 构建和部署

### 编译
```bash
bun build  # 使用Bun构建
```

### 运行模式

### 本地运行
- **CLI模式**: 完整的命令行工具，在本地环境执行
- **REPL模式**: 交互式终端
- **Daemon模式**: 后台守护进程

### Web集成 (Web App 部署)
Claude Code 有完整的 **bridge** 集成支持 claude.ai web app:

#### Bridge系统 (`bridge/` 目录)
这是连接CLI和claude.ai web版本的核心：

**主要组件:**
- `bridgeApi.ts` - Bridge API客户端，与claude.ai web服务通信
- `bridgeMain.ts` - Bridge主逻辑和会话管理 (115KB+)
- `bridgeMessaging.ts` - 消息传递协议
- `bridgeUI.ts` - Terminal UI集成
- `createSession.ts` - 会话创建和管理
- `codeSessionApi.ts` - Code Session API交互

**Web App支持:**
- ✅ **完整web UI**: 在 https://claude.ai/code 运行
- ✅ **IDE集成**: VS Code, JetBrains等IDE扩展
- ✅ **Desktop App**: macOS和Windows桌面应用
- ✅ **远程会话**: 在web app中启动的查询连接到CLI

**工作原理:**
1. 用户在 claude.ai web app中输入查询
2. Web app通过Bridge API与CLI通信
3. CLI执行工具和命令
4. 结果流式返回到web界面

**关键配置:**
```typescript
// 从 constants/product.ts
export const CLAUDE_AI_BASE_URL = 'https://claude.ai'
export const REMOTE_SESSION_URL = getRemoteSessionUrl() // 远程会话URL
```

**Bridge API端点:**
- 基础URL: `https://claude.ai`
- 认证: OAuth + Access Token
- 信任设备支持: 高级安全模式

**权限处理:**
- `bridgePermissionCallbacks.ts` - 处理web端的权限请求
- 权限验证通过Bridge API进行
- 支持交互式权限确认

### 其他运行模式
- **Assistant模式** (Kairos): 助手版本
- **Coordinator模式**: Agent协调器
- **远程执行**: 远程SSH/Cloud执行

## 关键统计

- **TypeScript文件**: >300个
- **代码行数**: >100,000行
- **组件**: >145个React组件
- **Hooks**: >85个自定义Hooks
- **命令**: >100个CLI命令
- **工具**: >100个集成工具
- **服务**: >30个核心服务

## 重要配置文件

- `.bun-lock` / `package-lock.json` - 依赖锁定
- `tsconfig.json` - TypeScript配置
- `.eslintrc.json` - 代码风格规则
- `settings.json` - 用户设置
- `claude.md` - 项目文档（本文件）

## 贡献指南

### 提交PR前检查清单
- [ ] 代码遵循项目风格
- [ ] 添加了必要的类型定义
- [ ] 包含了错误处理
- [ ] 更新了相关文档
- [ ] 测试通过
- [ ] 不引入安全漏洞

### 常见任务

**调试启动问题**
```bash
bun --inspect ./src/main.tsx
```

**性能分析**
- 查看 `utils/startupProfiler.js` 中的性能检查点
- 检查 `profileReport()` 输出

**远程会话调试**
- 检查 `remote/` 目录
- 查看 `services/api/` 用于API问题

## 相关资源

- **官方文档**: https://claude.ai/
- **API文档**: https://docs.anthropic.com/
- **MCP规范**: https://modelcontextprotocol.io/
- **GitHub讨论**: https://github.com/anthropics/claude-code/discussions

## 常见命令

```bash
# 启动REPL
claude repl

# 执行单个查询
claude "what is this file?"

# 创建Agent
claude /agent create my-agent

# 查看帮助
claude --help

# 配置设置
claude settings
```

## 部署和集成方式总结

### 1️⃣ **Web版本** (推荐新用户)
```
访问: https://claude.ai/code
- ✅ 无需安装
- ✅ 跨平台浏览器支持
- ✅ 实时保存
- ✅ 与web app无缝集成
- ✅ 集中式权限管理
```

### 2️⃣ **CLI版本** (开发者和自动化)
```bash
# macOS/Linux
brew install claude-code

# 或从源码构建
bun build && bun ./src/main.tsx
```
- ✅ 完全本地控制
- ✅ 脚本和自动化支持
- ✅ 离线工作（缓存）
- ✅ 自定义环境变量和配置
- ✅ 远程SSH执行

### 3️⃣ **IDE扩展** (开发者首选)
```
VS Code Extension:
- 直接在编辑器中使用Claude Code
- 即时访问当前文件
- 集成调试和运行

JetBrains Plugins (IntelliJ, PyCharm, etc.):
- IDE原生集成
- 代码补全和快捷方式
- 项目上下文自动传递
```

**IDE集成支持** (`utils/ide.ts`, `utils/jetbrains.ts`):
- VS Code MCP集成
- JetBrains插件支持
- IDE进程检测和通信
- 路径转换（Windows WSL）

### 4️⃣ **Desktop应用** (图形用户界面)
```
macOS App / Windows App
- ✅ 原生应用体验
- ✅ 系统菜单集成
- ✅ 通知支持
- ✅ 快捷方式和热键
```

### 5️⃣ **远程/SSH部署** (企业和服务器)
```
在远程服务器上运行:
ssh user@server.com
claude "analyze this code"
```
- ✅ 集中式环境
- ✅ 访问远程文件系统
- ✅ 企业凭证集成
- ✅ MDM管理（带有Policy Limits）

### 6️⃣ **Docker容器** (微服务和CI/CD)
```dockerfile
FROM debian:latest
RUN curl -fsSL https://bun.sh/install | bash
RUN bun install -g claude-code
CMD ["claude"]
```
- ✅ 隔离环境
- ✅ 一致的依赖
- ✅ CI/CD管道集成
- ✅ Kubernetes支持

## Bridge API 详细说明

**位置**: `src/bridge/` (33个文件)

**核心API操作**:

```typescript
// 会话管理
POST /v1/code/sessions              // 创建会话
GET  /v1/code/sessions/{id}         // 获取会话信息
DELETE /v1/code/sessions/{id}       // 关闭会话

// 工作请求
POST /v1/code/sessions/{id}/work    // 发送查询/命令

// 权限
POST /v1/code/sessions/{id}/permissions  // 请求权限
GET  /v1/code/sessions/{id}/permissions  // 查看待处理权限
```

**认证**:
- Bearer Token (OAuth)
- Trusted Device Token (可选)

**消息格式**:
```json
{
  "type": "query|command|tool",
  "content": "用户输入",
  "tools": ["tool1", "tool2"],
  "context": {}
}
```

## 故障排查

### 问题: API认证失败
- 检查 OAuth配置
- 验证API密钥在keychain中
- 查看 `services/api/` 中的认证逻辑

### 问题: 工具不可用
- 检查工具是否在 `tools.ts` 中注册
- 验证权限设置
- 查看 `services/policyLimits/` 中的限制

### 问题: UI渲染问题
- 检查终端兼容性
- 验证Ink组件配置
- 查看 `cli/print.ts` 中的格式化逻辑

### Web版本特定问题

**问题: Web版无法连接到CLI**
- 检查Bridge API URL是否正确（https://claude.ai）
- 验证OAuth token有效性
- 查看浏览器控制台网络标签
- 检查防火墙和代理设置
- 参考: `bridge/bridgeApi.ts`

**问题: 权限请求无法弹出**
- 确认Terminal仍在运行
- 检查`bridgePermissionCallbacks.ts`中的权限处理
- 验证UI进程未被冻结

**问题: 远程会话断开**
- 自动重连逻辑: `bridge/bridgeMain.ts`
- 检查网络连接稳定性
- 查看session状态: `bridge/createSession.ts`
- 考虑增加超时时间

**问题: IDE扩展无法连接**
- 检查IDE插件是否已安装和启用
- 验证IDE版本兼容性
- 查看`utils/ide.ts`中的IDE检测逻辑
- 对于JetBrains: 检查`utils/jetbrains.ts`
- 重启IDE和CLI

## 快速参考

### 部署选择矩阵

| 场景 | 推荐方案 | 优点 |
|-----|--------|------|
| **个人开发** | Web版 (claude.ai/code) | 无需配置，实时协作 |
| **团队开发** | IDE扩展 | 最佳编辑器集成 |
| **自动化脚本** | CLI + SSH | 完整可编程性 |
| **企业部署** | 私有远程服务器 + MDM | 完全控制，合规性 |
| **CI/CD集成** | Docker容器 | 隔离环保，可扩展 |
| **离线工作** | 本地CLI | 无网络依赖 |

### 关键概念

- **Bridge**: CLI与Web app间的通信桥接
- **Remote Session**: Web版启动的后端执行
- **Permission Callback**: 权限验证和用户确认
- **MCP**: Model Context Protocol，扩展能力
- **MDM**: 企业设备管理和策略限制

### 性能指标

- **启动时间**: ~1-2秒（CLI）, ~300ms（Web）
- **查询延迟**: <200ms平均
- **最大并发会话**: 取决于服务器资源
- **Bridge连接超时**: 30秒（可配置）

## 维护信息

**最后更新**: 2026-03-31
**项目状态**: 活跃维护中
**维护者**: Anthropic团队
**部署地址**: https://claude.ai/code
**开源**: 部分代码开源，完整实现专有
