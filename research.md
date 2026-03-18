# CC Switch 项目深度研究报告

## 1. 项目概述

**CC Switch** 是一个用 Tauri 2 构建的跨平台桌面应用程序，用于统一管理多种 AI 编程 CLI 工具的配置文件。

### 支持的工具

- **Claude Code** - Anthropic 的 AI 编程助手
- **Codex** - OpenAI 的 AI 编程助手
- **Gemini CLI** - Google 的 AI 编程 CLI
- **OpenCode** - 另一个 AI 编程 CLI 工具
- **OpenClaw** - 支持 agent 配置和 workspace 文件管理的 CLI

### 核心价值

- 解决不同 CLI 工具配置格式不一致的问题
- 提供可视化的 Provider 管理界面
- 支持一键切换 API 提供商
- 内置 50+ Provider 预设

## 2. 技术栈

### 前端

| 技术           | 版本  | 用途          |
| -------------- | ----- | ------------- |
| React          | 18.2  | UI 框架       |
| TypeScript     | 5.3+  | 类型系统      |
| Vite           | 7.3   | 构建工具      |
| TailwindCSS    | 3.4   | CSS 框架      |
| TanStack Query | 5.90  | 数据获取/缓存 |
| shadcn/ui      | -     | UI 组件库     |
| @dnd-kit       | 6.3   | 拖拽排序      |
| react-i18next  | 16.0  | 国际化        |
| framer-motion  | 12.23 | 动画          |
| recharts       | 3.5   | 图表          |
| codemirror     | 6.0   | 代码编辑器    |

### 后端 (Rust)

| 技术            | 版本 | 用途           |
| --------------- | ---- | -------------- |
| Tauri           | 2.8  | 桌面应用框架   |
| rusqlite        | 0.31 | SQLite 数据库  |
| axum            | 0.7  | HTTP 服务器    |
| tower           | 0.4  | 中间件框架     |
| hyper           | 1.0  | HTTP 实现      |
| tokio           | 1.0  | 异步运行时     |
| serde           | 1.0  | 序列化         |
| reqwest         | 0.12 | HTTP 客户端    |
| tauri-plugin-\* | 2.0  | Tauri 插件生态 |

### 测试

- **Frontend**: vitest + MSW + @testing-library/react
- **Backend**: cargo test + serial_test

## 3. 项目架构

### 3.1 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend (React + TS)                    │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │ Components  │  │    Hooks     │  │  TanStack Query  │    │
│  │   (UI)      │──│ (Bus. Logic) │──│   (Cache/Sync)   │    │
│  └─────────────┘  └──────────────┘  └──────────────────┘    │
└────────────────────────┬────────────────────────────────────┘
                         │ Tauri IPC
┌────────────────────────▼────────────────────────────────────┐
│                  Backend (Tauri + Rust)                     │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │  Commands   │  │   Services   │  │  Models/Config   │    │
│  │ (API Layer) │──│ (Bus. Layer) │──│     (Data)       │    │
│  └─────────────┘  └──────────────┘  └──────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 前端目录结构

```
src/
├── components/              # UI 组件
│   ├── providers/            # Provider 管理面板
│   ├── mcp/                 # MCP 服务器管理
│   ├── prompts/             # 提示词管理
│   ├── skills/              # Skills 管理
│   ├── sessions/            # 会话管理器
│   ├── proxy/               # 代理配置面板
│   ├── openclaw/            # OpenClaw 专用面板
│   ├── settings/             # 设置页面
│   ├── deeplink/            # Deep Link 导入
│   ├── env/                 # 环境变量管理
│   ├── universal/            # 跨应用配置
│   ├── usage/                # 用量统计
│   ├── workspace/            # Workspace 文件编辑器
│   └── ui/                  # shadcn/ui 基础组件
├── hooks/                   # 自定义 Hooks (业务逻辑)
├── lib/
│   ├── api/                 # Tauri API 封装
│   ├── query/               # TanStack Query 配置
│   ├── schemas/              # Zod 验证 schema
│   └── utils/               # 工具函数
├── locales/                  # 国际化文件 (zh/en/ja)
├── config/                   # 预设配置
│   ├── constants.ts
│   ├── appConfig.tsx
│   ├── claudeProviderPresets.ts
│   ├── codexProviderPresets.ts
│   ├── geminiProviderPresets.ts
│   ├── opencodeProviderPresets.ts
│   ├── openclawProviderPresets.ts
│   ├── universalProviderPresets.ts
│   ├── mcpPresets.ts
│   └── codexTemplates.ts
├── types/                   # TypeScript 类型定义
└── icons/                   # 图标资源
```

### 3.3 后端目录结构

```
src-tauri/src/
├── main.rs                  # 入口点
├── lib.rs                   # 库入口，App 配置和运行
├── commands/                # Tauri 命令层 (按领域组织)
│   ├── mod.rs
│   ├── provider.rs
│   ├── mcp.rs
│   ├── prompt.rs
│   ├── skill.rs
│   ├── proxy.rs
│   ├── settings.rs
│   ├── usage.rs
│   ├── session_manager.rs
│   ├── failover.rs
│   ├── deeplink.rs
│   └── ...
├── services/                # 业务逻辑层
│   ├── mod.rs
│   ├── ConfigService.rs
│   ├── ProviderService.rs
│   ├── McpService.rs
│   ├── PromptService.rs
│   ├── SkillService.rs
│   ├── ProxyService.rs
│   ├── SpeedtestService.rs
│   └── ...
├── database/                # SQLite DAO 层
│   ├── mod.rs
│   ├── schema.rs            # 表结构 + 迁移
│   ├── backup.rs            # 导入导出 + 快照
│   ├── migration.rs         # JSON → SQLite 迁移
│   └── dao/
│       ├── mod.rs
│       ├── providers.rs
│       ├── mcp.rs
│       ├── prompts.rs
│       ├── skills.rs
│       ├── settings.rs
│       ├── proxy.rs
│       ├── failover.rs
│       └── usage_rollup.rs
├── proxy/                   # 本地代理模块
│   ├── mod.rs
│   ├── server.rs            # Axum HTTP 服务器
│   ├── handlers.rs          # 请求处理器
│   ├── forwarder.rs          # 请求转发
│   ├── response_handler.rs   # 响应处理
│   ├── provider_router.rs    # Provider 路由
│   ├── circuit_breaker.rs    # 熔断器
│   ├── failover_switch.rs    # 故障转移切换
│   ├── model_mapper.rs       # 模型映射
│   ├── error_mapper.rs       # 错误转换
│   ├── body_filter.rs        # Body 过滤器
│   ├── cache_injector.rs     # 缓存注入
│   ├── thinking_optimizer.rs  # Thinking 优化
│   ├── thinking_rectifier.rs  # Thinking 修正
│   ├── health.rs             # 健康检查
│   ├── types.rs
│   ├── error.rs
│   ├── log_codes.rs
│   ├── http_client.rs
│   ├── handler_config.rs
│   ├── handler_context.rs
│   ├── session.rs
│   ├── response_processor.rs
│   ├── usage/
│   │   ├── mod.rs
│   │   ├── calculator.rs
│   │   ├── logger.rs
│   │   └── parser.rs
│   └── providers/
│       ├── mod.rs
│       ├── adapter.rs
│       ├── claude.rs
│       ├── codex.rs
│       ├── gemini.rs
│       ├── auth.rs
│       ├── streaming.rs
│       ├── streaming_responses.rs
│       ├── transform.rs
│       ├── transform_responses.rs
│       ├── copilot_auth.rs
│       └── models/
│           ├── mod.rs
│           ├── anthropic.rs
│           └── openai.rs
├── mcp/                     # MCP 同步模块
│   ├── mod.rs
│   ├── validation.rs
│   ├── claude.rs
│   ├── codex.rs
│   ├── gemini.rs
│   └── opencode.rs
├── deeplink/               # Deep Link 处理
│   ├── mod.rs
│   ├── parser.rs
│   ├── provider.rs
│   ├── mcp.rs
│   ├── skill.rs
│   ├── prompt.rs
│   ├── utils.rs
│   └── tests.rs
├── session_manager/         # 会话管理
├── app_config.rs            # 应用配置结构
├── app_store.rs             # 应用状态存储
├── config.rs                # 配置文件读写
├── provider.rs              # Provider 数据结构
├── settings.rs              # 设置管理
├── prompt_files.rs          # 提示词文件
├── error.rs                 # 错误类型
└── ... (其他模块)
```

## 4. 核心数据模型

### 4.1 Provider 结构

```rust
pub struct Provider {
    pub id: String,
    pub name: String,
    pub settings_config: Value,  // JSON 格式的设置
    pub website_url: Option<String>,
    pub category: Option<String>,  // 分类标签
    pub created_at: Option<i64>,
    pub sort_index: Option<usize>,
    pub notes: Option<String>,
    pub meta: Option<ProviderMeta>,  // 元数据
    pub icon: Option<String>,
    pub icon_color: Option<String>,
    pub in_failover_queue: bool,
}
```

### 4.2 ProviderMeta 元数据

```rust
pub struct ProviderMeta {
    pub custom_endpoints: HashMap<String, CustomEndpoint>,
    pub usage_script: Option<UsageScript>,
    pub endpoint_auto_select: Option<bool>,
    pub is_partner: Option<bool>,
    pub cost_multiplier: Option<String>,
    pub pricing_model_source: Option<String>,
    pub test_config: Option<ProviderTestConfig>,
    pub proxy_config: Option<ProviderProxyConfig>,
    pub api_format: Option<String>,  // "anthropic" | "openai_chat" | "openai_responses"
    pub auth_binding: Option<AuthBinding>,
    // ...
}
```

### 4.3 Universal Provider (跨应用共享)

```rust
pub struct UniversalProvider {
    pub id: String,
    pub name: String,
    pub provider_type: String,
    pub apps: UniversalProviderApps,  // { claude: bool, codex: bool, gemini: bool }
    pub base_url: String,
    pub api_key: String,
    pub models: UniversalProviderModels,
    // ...
}
```

### 4.4 MCP Server

```rust
pub struct McpServer {
    pub id: String,
    pub name: String,
    pub command: String,
    pub args: Vec<String>,
    pub env: HashMap<String, String>,
    pub apps: McpApps,  // 各应用的启用状态
    // ...
}
```

## 5. 核心功能模块

### 5.1 Provider 管理

- **CRUD 操作**: 添加、更新、删除 Provider
- **一键切换**: 切换当前激活的 Provider
- **排序**: 拖拽排序Provider列表
- **导入/导出**: 支持配置文件导入导出
- **Provider 预设**: 50+ 内置预设，覆盖各种 API 提供商

### 5.2 MCP 管理

- **统一面板**: 跨 4 个应用管理 MCP 服务器
- **双向同步**: 数据库 ↔ 各应用的 MCP 配置
- **导入功能**: 从 Claude/Codex/Gemini/OpenCode 导入现有配置
- **Deep Link 导入**: 通过 `ccswitch://` URL 导入

### 5.3 Skills 管理

- **统一存储**: SSOT (Single Source of Truth) 模式
- **仓库管理**: 支持添加自定义 GitHub 仓库
- **安装方式**: Symlink 或文件复制
- **迁移功能**: 从旧版本自动迁移

### 5.4 Prompts 管理

- **跨应用同步**: CLAUDE.md / AGENTS.md / GEMINI.md
- **Markdown 编辑器**: 带有语法高亮
- **回填保护**: 激活时备份现有文件

### 5.5 代理功能

#### 本地代理服务器

- 基于 Axum 构建的 HTTP 代理
- 支持请求/响应转换
- 模型映射 (Model Mapping)
- 认证信息注入

#### 故障转移 (Failover)

- 熔断器模式 (Circuit Breaker)
- 自动 failover 队列
- 健康检查和状态监控

#### 特殊处理

- **Thinking 优化**: 优化 Claude 的 thinking budget 使用
- **缓存注入**: 提高 token 效率
- **Body 过滤**: 过滤敏感信息

### 5.6 用量统计

- **请求日志**: 记录每次 API 请求
- **Token 统计**: 输入/输出 token 计数
- **趋势图表**: 展示用量变化
- **自定义定价**: 支持自定义模型价格

### 5.7 会话管理

- 浏览 Claude Code 的对话历史
- 搜索和恢复会话
- 跨应用会话管理 (OpenClaw)

## 6. 设计模式

### 6.1 SSOT (Single Source of Truth)

```
~/.cc-switch/cc-switch.db (SQLite)
```

所有数据存储在 SQLite 数据库，各应用的配置文件作为 "live" 文件。

### 6.2 双层存储

- **SQLite**: 可同步的数据 (Provider, MCP, Prompts, Skills)
- **JSON**: 设备级设置 (UI 偏好)

### 6.3 双向同步

- **写入**: Database → Live 文件
- **回填**: Live 文件 → Database (编辑当前激活 Provider 时)

### 6.4 原子写入

```
temp_file.json → rename → config.json
```

使用临时文件 + 重命名模式防止配置损坏。

### 6.5 分层架构

```
Commands (API Layer)
    ↓
Services (Business Layer)
    ↓
DAO (Data Access Layer)
    ↓
Database (SQLite)
```

## 7. 数据库 Schema

### 主要表

- `providers` - Provider 配置
- `universal_providers` - 统一 Provider
- `mcp_servers` - MCP 服务器
- `prompts` - 提示词
- `installed_skills` - 已安装 Skills
- `skill_repos` - Skill 仓库
- `settings` - 设置
- `stream_check_logs` - 流检查日志
- `usage_rollup` - 用量汇总
- `request_logs` - 请求日志

### Schema 版本

当前版本: **6**

迁移时会自动创建备份。

## 8. Deep Link

### URL 格式

```
ccswitch://import/provider?data=base64(JSON)
ccswitch://import/mcp?data=base64(JSON)
ccswitch://import/skill?url=...
ccswitch://import/prompt?data=base64(JSON)
```

### 处理流程

1. 解析 URL
2. 发射 `deeplink-import` 事件到前端
3. 前端显示确认对话框
4. 用户确认后执行导入

## 9. 系统集成

### 9.1 系统托盘

- 动态托盘菜单
- 显示当前 Provider
- 快速切换

### 9.2 自动启动

- 支持开机自启动
- 静默启动模式

### 9.3 云同步

- WebDAV 服务器
- 支持 Dropbox, OneDrive, iCloud
- 自动同步配置变更

### 9.4 深链接

- macOS AppleEvent
- Windows/Linux 命令行参数
- 自定义 URL 协议 (`ccswitch://`)

## 10. 开发指南

### 环境要求

- Node.js 18+
- pnpm 8+
- Rust 1.85+
- Tauri CLI 2.8+

### 开发命令

```bash
pnpm install          # 安装依赖
pnpm dev              # 开发模式 (热重载)
pnpm build            # 构建
pnpm typecheck        # 类型检查
pnpm format           # 格式化代码
pnpm test:unit        # 前端单元测试

cd src-tauri
cargo build           # Rust 构建
cargo test            # Rust 测试
cargo clippy          # Rust 代码检查
```

### 测试框架

- **Frontend**: vitest + MSW (Mock Service Worker)
- **Backend**: cargo test + serial_test

## 11. 关键配置文件路径

| 路径                         | 说明          |
| ---------------------------- | ------------- |
| `~/.cc-switch/cc-switch.db`  | SQLite 数据库 |
| `~/.cc-switch/settings.json` | 本地设置      |
| `~/.cc-switch/backups/`      | 自动备份      |
| `~/.cc-switch/skills/`       | Skills 文件   |
| `~/.cc-switch/logs/`         | 日志文件      |

## 12. 版本信息

- **当前版本**: 3.12.3
- **许可证**: MIT
- **作者**: Jason Young
- **仓库**: github.com/farion1231/cc-switch

## 13. 总结

CC Switch 是一个功能完善的 AI 编程 CLI 管理工具，采用了现代化的技术栈和成熟的架构设计。其核心优势包括：

1. **统一管理**: 一个应用管理 5 种不同的 CLI 工具
2. **热切换**: 支持 Provider 热切换，无需重启终端
3. **故障转移**: 内置熔断器和自动故障转移
4. **数据安全**: SQLite 原子写入 + 自动备份
5. **跨平台**: 原生桌面应用，支持 Windows/macOS/Linux
6. **可扩展**: 预留了丰富的插件和集成接口
