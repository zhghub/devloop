# 🏗️ DevLoop 架构 (一图流)

> 一图 + 一表 + 一句话,看完懂 DevLoop 怎么工作。
> 详细架构看 [DEVLoop-V2.3-MASTER.md](DEVLoop-V2.3-MASTER.md)

## 🎯 一句话

> DevLoop = **数据底座** (PostgreSQL) + **8 角色协作** (ACP 协议) + **代际进化审查** (机器学习式审计)

## 🏛️ 三层架构

```
┌─────────────────────────────────────────────────────────────┐
│  👤 用户层                                                  │
│  开发者 (你) / 飞书 / 企微 / 微信 / 管理后台                  │
└────────────────────────┬────────────────────────────────────┘
                         ↓ 方案/任务/API 调用
┌─────────────────────────────────────────────────────────────┐
│  🎼 调度层 (DevLoop Backend)                                │
│  Fastify + Node.js 22 + 196 REST API                        │
│  - 任务总线   - 智能审查 (代际)   - 经验库                  │
│  - 铁律系统   - 论坛/公告        - 仪表盘                   │
└────────────────────────┬────────────────────────────────────┘
                         ↓ ACP 协议 (JSON-RPC over stdio)
┌─────────────────────────────────────────────────────────────┐
│  🤖 Agent 层 (8 角色协作)                                   │
│  leader / intake / architect / reviewer /                    │
│  developer / auditor / assistant / doc                      │
│  ↓ 调用外部 AI 工具                                          │
│  Claude / Codex / Gemini / MiniMax / ... (12 个 harness)     │
└─────────────────────────────────────────────────────────────┘
                         ↓ 读写文件 / git commit
┌─────────────────────────────────────────────────────────────┐
│  📦 你的 Git 项目                                            │
└─────────────────────────────────────────────────────────────┘
```

## 📊 核心数据流 (任务派发闭环)

```
1️⃣ 方案接入 (4 入口)
   管理后台 / 飞书 / 企微 / 微信 → POST /intake
        ↓
2️⃣ 方案解析
   intake-agent → plan-parsing-agent → 拆解模块/任务
        ↓
3️⃣ 任务入队
   INSERT INTO dev_task_logs → INSERT INTO dev_spawn_queue
        ↓
4️⃣ 任务派发 (PG NOTIFY 事件驱动)
   dev_spawn_queue NOTIFY → 守护进程 LISTEN → claim
        ↓ HTTP wake-dispatch
5️⃣ Agent 执行 (ACP 协议)
   OpenClaw sessions_spawn(runtime="acp") → 选 1 个 harness
        ↓ JSON-RPC over stdio NDJSON
6️⃣ 真改文件 + 真 commit
   Agent 写代码 → git commit -m "..."
        ↓
7️⃣ 复盘回写 (POST /spawn-result)
   status=completed + commit_hash + files_changed
        ↓
8️⃣ 自动 lessons / 铁律更新
   spawn-result §6 触发 → dev_lessons_learned
        ↓
9️⃣ 代际审查 (generation + 1)
   review-agent 读上次 generation 数据 → 调整本轮策略
```

## 🧩 8 角色职责表

| # | 角色 | Agent | 职责 |
|---|---|---|---|
| 1 | **leader** | openclaw | 调度中枢,管人/管活/管质量/管表盘/管通信 |
| 2 | **intake** | openclaw-intake | 4 入口方案识别 |
| 3 | **plan-parsing** | openclaw-plan | 方案 → 模块/任务拆解 |
| 4 | **reviewer** | codex | 代际进化审查 (5 维: 安全/正确/性能/可维护/合规) |
| 5 | **developer** | claude-main | 领任务、写代码、commit |
| 6 | **auditor** | mini-agent | 安全扫描 / 深度审计 |
| 7 | **assistant** | opencode | 辅助 (片段/REPL) |
| 8 | **doc** | openclaw-doc | 自动写 lessons/铁律/复盘 |

> 详细 6 流程 (A 接入 → F 自演化) 看 [DEVLOG-WORKFLOW-V3.md](DEVLOG-WORKFLOW-V3.md)

## 🗄️ 16 张核心表

| 域 | 表 |
|---|---|
| 任务 | dev_task_logs / dev_spawn_queue / dev_spawn_request |
| 经验 | dev_lessons_learned / dev_guidelines / dev_decisions |
| Agent | dev_agent_registry / dev_agent_activity / dev_agent_positions |
| 论坛 | dev_announcements / dev_announcement_replies / dev_announcement_reactions |
| 项目 | project_modules / project_versions / project_changelog |
| 审计 | dev_audit_runs / dev_alert_history |

> 完整 ER 图: [devlog-ER.md](devloop-ER.md)

## 🔌 12 ACP Harness 兼容

| 类别 | Harness |
|---|---|
| **已支持** | openclaw (内置) |
| **2 档 (需 npm install)** | claude / codex / opencode / gemini |
| **3 档 (后续)** | copilot / cursor / droid / iflow / kilocode / kimi / kiro / qwen |

> 详细: [ACP-AGENT-CAPABILITY-MATRIX.md](ACP-AGENT-CAPABILITY-MATRIX.md)

## 🎯 三大核心能力

| 能力 | 实现 | 文档 |
|---|---|---|
| **📥 4 入口方案接入** | intake-agent + plan-parsing-agent | [DEVLOG-WORKFLOW-V3 §4.1](DEVLoop-WORKFLOW-V3.md) |
| **🧠 代际进化审查** | dev_audit_runs.generation + review.js prompt | [DEVLOG-WORKFLOW-V3 §4.3](DEVLoop-WORKFLOW-V3.md) |
| **🔧 自修复闭环** | findings → tasks → developer → 验证 → 下一代审查 | [DEVLOG-WORKFLOW-V3 §4.3-4.4](DEVLoop-WORKFLOW-V3.md) |

## 📦 技术栈

| 层 | 技术 |
|---|---|
| 后端框架 | Fastify 4 |
| 数据库 | PostgreSQL 15 |
| 缓存 | Redis 7 |
| ORM | 原生 pg (不用 Prisma) |
| AI 协议 | ACP (Agent Client Protocol) v1 |
| 部署 | Docker Compose |
| 监控 | 自建 workboard + Grafana (可选) |

---

🔄 [devloop.cn](https://devloop.cn) · [GitHub](https://github.com/zhghub/devloop)
