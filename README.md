<p align="center">
  <a href="https://shittycodingagent.ai">
    <img src="https://shittycodingagent.ai/logo.svg" alt="pi logo" width="128">
  </a>
</p>
<p align="center">
  <a href="https://discord.com/invite/3cU7Bz4UPx"><img alt="Discord" src="https://img.shields.io/badge/discord-community-5865F2?style=flat-square&logo=discord&logoColor=white" /></a>
  <a href="https://github.com/badlogic/pi-mono/actions/workflows/ci.yml"><img alt="Build status" src="https://img.shields.io/github/actions/workflow/status/badlogic/pi-mono/ci.yml?style=flat-square&branch=main" /></a>
</p>
<p align="center">
  <a href="https://pi.dev">pi.dev</a> domain graciously donated by
  <br /><br />
  <a href="https://exe.dev"><img src="packages/coding-agent/docs/images/exy.png" alt="Exy mascot" width="48" /><br />exe.dev</a>
</p>

# Pi Monorepo

> **Looking for the pi coding agent?** See **[packages/coding-agent](packages/coding-agent)** for installation and usage.

Tools for building AI agents and managing LLM deployments.

## Packages

| Package | Description |
|---------|-------------|
| **[@mariozechner/pi-ai](packages/ai)** | Unified multi-provider LLM API (OpenAI, Anthropic, Google, etc.) |
| **[@mariozechner/pi-agent-core](packages/agent)** | Agent runtime with tool calling and state management |
| **[@mariozechner/pi-coding-agent](packages/coding-agent)** | Interactive coding agent CLI |
| **[@mariozechner/pi-mom](packages/mom)** | Slack bot that delegates messages to the pi coding agent |
| **[@mariozechner/pi-tui](packages/tui)** | Terminal UI library with differential rendering |
| **[@mariozechner/pi-web-ui](packages/web-ui)** | Web components for AI chat interfaces |
| **[@mariozechner/pi-pods](packages/pods)** | CLI for managing vLLM deployments on GPU pods |

## 模块分析（中文）

### 总体拆分逻辑

项目按“核心能力 → 运行时 → 交互界面/产品化”分层，保证能力复用与产品形态解耦：

- **pi-ai** 负责多模型/多厂商的统一调用能力。
- **pi-agent-core** 在上层封装会话、工具调用和事件流。
- **pi-tui** 提供终端 UI 基础设施。
- **pi-coding-agent / pi-web-ui / pi-mom / pi-pods** 面向具体场景（CLI、Web、Slack、GPU Pod 管理）。

### 模块职责、关系与扩展点

#### @mariozechner/pi-ai（packages/ai）
- **功能**：统一 LLM API、模型发现、工具调用事件流、token/成本统计、跨提供商上下文承接。
- **为何拆分**：把“模型调用”从“Agent 逻辑”中独立出来，降低上层产品对特定厂商的耦合。
- **关系**：被 `pi-agent-core` 与 `pi-web-ui`、`pi-coding-agent`、`pi-mom` 间接复用。
- **扩展点**：
  - 自定义模型（自定义 `Model`，支持 OpenAI 兼容 API）。
  - 通过 `baseUrl`/`compat` 连接自建或代理服务。
  - Tool schema（TypeBox）与工具调用验证。

#### @mariozechner/pi-agent-core（packages/agent）
- **功能**：有状态 Agent 运行时（消息、工具、事件流、重试/中断），对接 `pi-ai`。
- **为何拆分**：将“对话编排/工具执行”从 UI 和产品中抽离，可供 CLI/Web/Slack 复用。
- **关系**：依赖 `pi-ai`；被 `pi-coding-agent`、`pi-web-ui`、`pi-mom`、`pi-pods` 使用。
- **扩展点**：
  - 自定义消息类型（声明合并）与 `convertToLlm` 转换。
  - `transformContext` 做裁剪/压缩/外部上下文注入。
  - 自定义工具（`AgentTool`）与自定义 `streamFn`。

#### @mariozechner/pi-tui（packages/tui）
- **功能**：终端 UI 渲染框架（差分渲染、组件、输入、Overlay）。
- **为何拆分**：终端 UI 能力与 Agent/模型逻辑独立，便于 CLI 与其他终端工具复用。
- **关系**：`pi-coding-agent` 直接依赖；`pi-web-ui` 复用部分文本/渲染工具。
- **扩展点**：
  - 自定义组件（`Component`/`Focusable`）。
  - 自定义主题、Overlay 布局、输入处理。

#### @mariozechner/pi-coding-agent（packages/coding-agent）
- **功能**：交互式编码 Agent CLI（会话管理、工具调用、TUI、扩展系统）。
- **为何拆分**：面向开发者的核心产品形态，与底层 Agent/模型分离。
- **关系**：依赖 `pi-agent-core`、`pi-ai`、`pi-tui`；被 `pi-mom` 复用。
- **扩展点**：
  - Extensions/Skills/Prompt Templates/Themes（`.pi` 目录或全局目录）。
  - Pi Packages（通过 npm/git 发布扩展）。
  - 配置文件（`settings.json`、`AGENTS.md`、`SYSTEM.md`）。

#### @mariozechner/pi-mom（packages/mom）
- **功能**：Slack Bot，将消息委派给 Agent，支持工具执行与持久工作区。
- **为何拆分**：Slack 交互与权限模型独立于 CLI；便于安全隔离与多实例部署。
- **关系**：依赖 `pi-agent-core`、`pi-ai`、`pi-coding-agent`。
- **扩展点**：
  - 自定义技能（工作区 `skills/`）。
  - 事件系统（定时/周期唤醒）。
  - Docker/Host sandbox 与工作区结构可定制。

#### @mariozechner/pi-pods（packages/pods）
- **功能**：管理 GPU Pod 与 vLLM 部署，提供 OpenAI 兼容接口与测试 Agent。
- **为何拆分**：模型部署运维逻辑与 Agent/UI 解耦，便于独立发布与迭代。
- **关系**：依赖 `pi-agent-core`；与 `pi-ai` 生态互补（模型调用端 vs 模型托管端）。
- **扩展点**：
  - 自定义模型与 vLLM 启动参数（`--vllm`）。
  - 扩展模型清单与部署策略（`models.json`/文档）。

#### @mariozechner/pi-web-ui（packages/web-ui）
- **功能**：Web 端聊天 UI 组件（消息流、附件、Artifacts、存储、代理）。
- **为何拆分**：Web UI 形态与 CLI/TUI 分离，便于集成到浏览器/产品端。
- **关系**：使用 `pi-agent-core` 的事件模型，结合 `pi-ai` 的模型与工具；与 `pi-tui` 共享部分文本/渲染工具。
- **扩展点**：
  - 自定义工具与工具渲染器（Tool Renderer）。
  - 自定义消息类型与 `convertToLlm`。
  - 存储后端、CORS 代理与自定义 Provider。

### 模块关系概览

```
pi-ai ──▶ pi-agent-core ──▶ pi-coding-agent ──▶ pi-mom
   │             │
   │             ├──▶ pi-web-ui
   │             └──▶ pi-pods
   └──▶ (被所有上层模块间接复用)

pi-tui ──▶ pi-coding-agent
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines and [AGENTS.md](AGENTS.md) for project-specific rules (for both humans and agents).

## Development

```bash
npm install          # Install all dependencies
npm run build        # Build all packages
npm run check        # Lint, format, and type check
./test.sh            # Run tests (skips LLM-dependent tests without API keys)
./pi-test.sh         # Run pi from sources (must be run from repo root)
```

> **Note:** `npm run check` requires `npm run build` to be run first. The web-ui package uses `tsc` which needs compiled `.d.ts` files from dependencies.

## License

MIT
