# ACP Agent 能力矩阵（小辉数字员工）

> 编制时间：2026-06-11 20:55
> 编制人：OpenClaw main (海刚指令 "全面了解后再动手")
> 数据源：每个 agent 的**官方文档**（不是 acpx 文档）

## 0. 核心认知：什么是 "ACP 调度"？

**OpenClaw 自带完整 ACP 调度机制**（`docs.openclaw.ai/tools/acp-agents`）：

```
飞书/企微消息
  → OpenClaw Gateway (acp.dispatch 自动路由)
  → @openclaw/acpx 插件 (官方 runtime)
  → 外部 ACP agent (codex/claude/opencode/...)
```

**小辉（OpenClaw 大脑）只需要用 `sessions_spawn` 工具派发任务**，不需要写任何 Python/Node 调度脚本。

## 1. ACP 协议核心方法（agentclientprotocol.com v1）

所有 ACP agent 必须实现这些 JSON-RPC 方法：

| 方法 | 方向 | 作用 |
|---|---|---|
| `initialize` | Client → Agent | 协商 protocolVersion + capabilities |
| `session/new` | Client → Agent | 创建新会话 |
| `session/load` | Client → Agent | 加载已有会话 |
| `session/prompt` | Client → Agent | 发送用户消息 |
| `session/cancel` | Client → Agent | 取消当前 turn |
| `session/update` | Agent → Client | 流式返回消息更新 |
| `session/list` | Client → Agent | 列出会话（可选） |
| `session/resume` | Client → Agent | 恢复会话（可选） |
| `session/close` | Client → Agent | 关闭会话（可选） |
| `authenticate` | Client → Agent | 认证（如果需要） |
| `fs/read_text_file` | Agent → Client | Agent 请求读文件（需 client 支持） |
| `fs/write_text_file` | Agent → Client | Agent 请求写文件 |
| `terminal/*` | Agent → Client | Agent 请求执行 shell |

**Capabilities 协商**：
- Client capabilities: `fs.readTextFile / fs.writeTextFile / terminal`
- Agent capabilities: `loadSession / promptCapabilities{image,audio,embeddedContext} / mcpCapabilities / auth`

## 2. 12 个 ACP Harness 能力对比

| Harness | 官方仓库 | 安装方式 | Auth | 备注 |
|---|---|---|---|---|
| **claude** | `agentclientprotocol/claude-agent-acp` | `npx -y @agentclientprotocol/claude-agent-acp` | Anthropic API key / OAuth | 基于 Claude Agent SDK |
| **codex** | `agentclientprotocol/codex-acp` | `npx -y @agentclientprotocol/codex-acp` | OpenAI API key | 隔离 CODEX_HOME |
| **copilot** | GitHub Copilot ACP | 内置 | GitHub auth | |
| **cursor** | `cursor-agent acp` | 内置 | Cursor auth | 命令是 `cursor-agent acp` |
| **droid** | Factory Droid CLI | 内置 | `FACTORY_API_KEY` | |
| **gemini** | `google-gemini/gemini-cli` | `npx @google/gemini-cli` | Google OAuth / API key | |
| **iflow** | iFlow CLI | 内置 | iFlow auth | |
| **kilocode** | Kilo Code CLI | `npx -y @kilocode/cli acp` | Kilo auth | |
| **kimi** | Kimi CLI | `kimi acp` | Moonshot auth | |
| **kiro** | Kiro CLI | `kiro-cli-chat acp` | AWS auth | |
| **opencode** | `opencode-ai` | `npx -y opencode-ai acp` | OpenCode auth | |
| **qwen** | Qwen CLI | `qwen --acp` | Qwen auth | |
| **openclaw** | OpenClaw Gateway bridge | `openclaw acp` | 内置 | IDE→OpenClaw 入向 |

## 3. 主机真实状态（2026-06-11 21:00）

| Agent | 是否真安装 | 备注 |
|---|---|---|
| **claude** | ❌ | acpx 配置 `command: claude`，但主机没装 |
| **codex** | ❌ | `codex-acp` 找不到 |
| **opencode** | ❌ | `opencode-ai` 找不到 |
| **gemini** | ❌ | `@google/gemini-cli` 没装 |
| **openclaw** | ✅ | 内置 `openclaw acp` 命令 |
| **kimi / kiro / qwen** | ❌ | 没装 |
| **droid / copilot / cursor** | ❌ | 没装 |

**结论**：12 个 harness 理论上都支持，**但主机上只有 `openclaw` 1 个能跑**。
**这是不是真的？需要实跑 `/acp doctor` 验证**。

## 4. OpenClaw ACP 配置现状

`/root/.openclaw/openclaw.json` 当前 ACP 段：
```json
{
  "enabled": true,
  "backend": "acpx",
  "defaultAgent": "claude",
  "allowedAgents": [
    "claude", "codex", "opencode", "gemini", "cursor",
    "claude-inspector", "claude-fixer", "claude-architect", "claude-developer"
  ],
  "maxConcurrentSessions": 4,
  "fallbacks": ["codex", "gemini", "cursor"]
}
```

**问题**：
1. `defaultAgent: "claude"` 但主机没装 → 失败
2. `allowedAgents` 里有 `claude-inspector` / `claude-fixer` / `claude-architect` / `claude-developer` — **这些不是 acpx 内置 harness**！acpx 只认 `claude` / `codex` / `opencode` 等基础名
3. `acp.dispatch.enabled` 没显式写（默认 true）
4. `runtime.ttlMinutes` 没配（默认 120）

## 5. 我能用什么（推荐优先级）

### 第一档（主机已装，必跑通）
- **openclaw**（`openclaw acp` 内置）— 入向：让外部 IDE 通过 ACP 接 OpenClaw

### 第二档（要 npm install 后才能跑）
- **codex** / **claude** / **opencode** / **gemini** — 需要先 `npm i -g @agentclientprotocol/codex-acp` 等

### 第三档（暂缓，先把第二档搞通）
- 其他 8 个 harness

## 6. OpenClaw `sessions_spawn` 工具参数（我直接调的工具）

```typescript
sessions_spawn({
  task: "string",           // 必填，发送给 ACP 会话
  runtime: "acp",           // 必填，必须 "acp"
  agentId: "codex",         // 可选，默认 acp.defaultAgent
  thread: true,             // 可选，请求话题绑定
  mode: "session",          // 可选：run（一次性）/ session（持久）
  cwd: "/path/to/repo",     // 可选，工作目录
  label: "backend",         // 可选，操作员可见标签
  streamTo: "parent"        // 可选，初始运行进度流回请求者
})
```

**重要约束**（来自 docs）：
- ❌ 沙盒化会话不能 spawn ACP（`runtime: "acp"` 在主机端跑）
- ❌ `sandbox: "require"` 与 `runtime: "acp"` 不兼容
- ✅ 主会话（非沙盒）才能用
- ✅ `mode: "session"` 需要 `thread: true`

## 7. 权限配置（acpx 插件）

`/root/.openclaw/openclaw.json` → `plugins.entries.acpx.config`：

```json
{
  "permissionMode": "approve-all",       // approve-all / approve-reads / deny-all
  "nonInteractivePermissions": "fail",  // fail（默认）/ deny
  "timeoutSeconds": 120,                // 启动和初始化超时
  "probeAgent": "codex"                 // /acp doctor 用哪个 agent 探测
}
```

**当前状态**：`acpx` 插件**没在 `plugins.entries` 里**！OpenClaw 用的是 acpx 0.6.1 全局 CLI，不是 `@openclaw/acpx` 插件。

**关键问题**：OpenClaw 的 acp.backend 是 "acpx" → 走全局 acpx CLI（0.6.1），而不是 OpenClaw 官方 `@openclaw/acpx` 插件。
**两个有差别吗？** 待查。可能是同一个，只是 OpenClaw 把 npm 全局装的 acpx 0.6.1 当作 backend。

## 8. 真测试建议（30 分钟后海刚拍板）

**测试 1：`/acp doctor` 看 backend 状态**
```bash
openclaw acp doctor
# 或
/acp doctor
```

**测试 2：用 `sessions_spawn` 跑 1 个真实任务（不写脚本）**
```javascript
// 在 OpenClaw 工具调用里直接发
sessions_spawn({
  task: "echo hello from acp dispatch",
  runtime: "acp",
  agentId: "openclaw",  // 唯一确定能跑的
  mode: "run"
})
```

**测试 3：装一个真实 harness（npm install -g）后跑**
```bash
npm install -g @agentclientprotocol/codex-acp
# 然后在 OpenClaw 改 acp.allowedAgents 加 codex
# 再 sessions_spawn agentId="codex"
```

## 9. 与之前理解的差异（永久铁律）

| 之前的我 | 现在的我 |
|---|---|
| ❌ 写 Python 脚本 dispatch.py / spawn-consumer-*.py 调度 | ✅ 永不写脚本，OpenClaw + sessions_spawn |
| ❌ 用 subprocess.run(acpx ...) 绕路 | ✅ sessions_spawn 是 OpenClaw 原生工具 |
| ❌ V5/V6 后台 consumer 循环 | ✅ OpenClaw acp.dispatch 自动接管 |
| ❌ acpx CLI 当主入口 | ✅ acpx 是底层，OpenClaw 是大脑 |

**永久铁律已写进 MEMORY.md**
