# 多部门多 Agent 协同系统 V5.0 — OpenClaw 原生 Agent + sessions_send 通信

> 任务: T2026-0509-010
> 版本: V5.0 (2026-06-17)
> 状态: Phase1 落地 / Phase2-3 设计稿
> 适用: OpenClaw 多 Agent 协作场景

---

## 一、问题

### 1.1 历史背景
V4.0 之前, 团队曾以 `sessions_spawn` 串接子任务, 实际跑下来发现两个长期阻塞点:

1. **V3 范式陈旧**: 调度中心早期版本使用 `scripts/v5-scheduler.py`, 2026-06-11 后被 `b54e0227` 清掉, 改走 OpenClaw 网关 + systemd `devlog-pg-listen.service` (NOTIFY 事件驱动). 任何依赖 `sessions_spawn` 串接的方案都按 V3 范式重写.
2. **V4.0 废弃原因不成立**: 任务 issues 明确指出"`sessions_spawn` 限制是针对 Claude Code subagent, 不是 OpenClaw 原生 Agent" — 这条误解曾导致整个 V4 搁置, 实际验证发现 `sessions_send` 在两个 OpenClaw 原生 Agent 之间可以直连.

### 1.2 真实痛点
当前仅 `main` 与 `xiaohui-2-0-dev` 两个原生 Agent. 业务部门 (销售/客服/仓配) 任务全都回灌 `claude-main`, 串行处理导致:
- 单个 devlog 任务被多个部门需求串行阻塞 (P0→P1→P2, 实际部门可并行)
- Agent 上下文跨任务污染 (一个 Agent 临时记忆丢失/不完整)
- 跨部门任务的知识中心缺失, 每次 dispatch 都要重新加载上下文

### 1.3 V5.0 目标
建立"OpenClaw 原生 Agent + sessions_send"的多部门并行协同系统, 解决:
- **部门隔离**: 销售/客服/仓配各一个独立 Agent, 各自 workspace
- **知识中心**: 外部 PostgreSQL Session Memory API, Agent 临时记忆持久化
- **路由分发**: before_dispatch hook 按部门路由, 不再全回 `claude-main`
- **极速模式**: 简单任务 (单部门/单模块) 走快速通道, 不启动完整 dispatch

---

## 二、目标

### 2.1 业务目标
| 目标 | 量化指标 | 验证方法 |
|------|----------|----------|
| 部门任务并行化 | 跨部门任务 P95 派发延迟 ≤ 30s | 任务 dispatch 时间戳 diff |
| Agent 上下文隔离 | 部门 Agent 工作区无交叉写入 | workspace tree diff |
| 知识复用 | department_knowledge 命中 ≥ 60% | API access log |
| 极速模式覆盖 | 单部门任务走 fast path 比例 ≥ 40% | dispatch metrics |

### 2.2 技术目标
- 在 `/root/.openclaw/openclaw.json` 的 `agents.list` 注册 3 个部门 Agent (xs/cc/cw)
- 每个 Agent 独立 workspace: SOUL.md + AGENTS.md + memory/
- `before_dispatch` hook 按 `department` 字段路由到对应 Agent
- PostgreSQL `department_knowledge` 表 + CRUD API 落地外部记忆中心

### 2.3 不做
- 不复活 `sessions_spawn` (V3 范式, 已被清)
- 不改 OpenClaw 网关 (27747) 协议层
- 不引入新的 agent runtime (复用 OpenClaw 原生)

---

## 三、修复方案

### 3.1 架构总览

```
                ┌──────────────────────────────────────┐
                │       OpenClaw Gateway (27747)       │
                └──────────────────┬───────────────────┘
                                   │
                  ┌────────────────┴────────────────┐
                  │       before_dispatch hook       │
                  │  (按 task.department 路由)       │
                  └────────────────┬────────────────┘
                                   │
        ┌──────────┬───────────────┼───────────────┬──────────┐
        ▼          ▼               ▼               ▼          ▼
    ┌──────┐  ┌──────┐        ┌──────┐        ┌──────┐  ┌──────┐
    │ xs   │  │ cc   │        │ cw   │        │main  │  │xiaohui│
    │销售部│  │客服部│        │仓配部│        │协调 │  │执行  │
    └──┬───┘  └──┬───┘        └──┬───┘        └──┬───┘  └──┬───┘
       │         │               │               │         │
       └─────────┴───────────────┴───────────────┴─────────┘
                                   │
                  ┌────────────────┴────────────────┐
                  │     PostgreSQL Session Memory    │
                  │  ┌──────────────────────────┐   │
                  │  │ department_knowledge     │   │
                  │  │ session_memory           │   │
                  │  │ dispatch_log             │   │
                  │  └──────────────────────────┘   │
                  └──────────────────────────────────┘
```

### 3.2 Phase 1: 注册部门 Agent (本任务交付)

#### 3.2.1 openclaw.json 改动

`/root/.openclaw/openclaw.json` 的 `agents.list` 由 2 项扩展为 5 项:

```json
"list": [
  { "id": "main",                "workspace": "/root/.openclaw/workspace" },
  { "id": "xiaohui-2-0-dev",     "workspace": "/root/.openclaw/workspace/agents/xiaohui-2-0-dev" },
  { "id": "xs-agent",            "workspace": "/root/.openclaw/workspace/agents/xs-agent" },
  { "id": "cc-agent",            "workspace": "/root/.openclaw/workspace/agents/cc-agent" },
  { "id": "cw-agent",            "workspace": "/root/.openclaw/workspace/agents/cw-agent" }
]
```

**注意**:
- 三个部门 Agent **不加进** `acp.allowedAgents` (后者只约束 Claude Code 等外部 subagent, 不影响原生 Agent 互发 sessions_send)
- 部门 Agent 复用 `agents.defaults` (maxConcurrent=4, model=MiniMax-M3) — 不需要单独配 model

#### 3.2.2 Workspace 模板

每个部门 Agent workspace 包含 4 件套:

```
agents/<dept>-agent/
├── SOUL.md          # Agent 人格: 部门使命 + 边界 + 与其他部门的关系
├── AGENTS.md        # OpenClaw workspace 标准 AGENTS.md (复用 xiaohui-2-0-dev)
├── memory/          # 长期记忆目录, MEMORY.md + 每日笔记
└── CLAUDE.md        # 部门范围说明 + 不修改范围
```

**SOUL.md 模板要点** (以 xs 销售部为例):
- 核心使命: 处理销售订单/客户/报价/分销相关 dispatch
- 边界: 不直接改 inventory/printer/logistics 模块 (转 cw 或 cc)
- 协作: 收到 dispatch 涉及仓配 → sessions_send 给 cw-agent
- 极速模式: 单销售模块 (如 P2 报价模板) 不走 dispatch, 直接 commit

#### 3.2.3 before_dispatch 路由 hook

实现位置: `/root/.openclaw/workspace/coordinator/before-dispatch.js` (新建)

```js
// before-dispatch.js
// 2026-06-17 海刚 V5.0 Phase1
// 路由规则: 按 task.department 字段映射到目标 Agent
const DEPT_TO_AGENT = {
  sales:   'xs-agent',
  service: 'cc-agent',
  warehouse:'cw-agent',
  default: 'main',
};

function routeDispatch(task) {
  const agent = DEPT_TO_AGENT[task.department] || DEPT_TO_AGENT.default;
  return { ...task, routedTo: agent };
}
```

钩入位置: coordinator 的 dispatch loop (待 V5.0 Phase 1 续接, 6/20 前完成)

### 3.3 Phase 2: 部门知识中心 (后续任务)

数据库 schema:

```sql
CREATE TABLE department_knowledge (
  id           SERIAL PRIMARY KEY,
  department   VARCHAR(32) NOT NULL,  -- sales/service/warehouse
  category     VARCHAR(64) NOT NULL,  -- product-policy / customer-faq / sku-mapping
  content      JSONB       NOT NULL,
  version      INT         DEFAULT 1,
  updated_by   VARCHAR(64),           -- 哪个 Agent 写入
  created_at   TIMESTAMPTZ DEFAULT NOW(),
  updated_at   TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(department, category)
);

CREATE TABLE session_memory (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id     VARCHAR(64) NOT NULL,
  session_id   VARCHAR(64) NOT NULL,
  payload      JSONB       NOT NULL,
  expires_at   TIMESTAMPTZ,
  created_at   TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_session_memory_agent_session ON session_memory(agent_id, session_id);
```

CRUD API: `backend/src/modules/devlog/routes/knowledge.js` (后续任务)

### 3.4 Phase 3: 极速模式 (后续任务)

判定逻辑 (before-dispatch):
1. 任务 `module` 在部门 Agent 已知模块白名单 (xs 销售: order/customer/quote/distribution)
2. 任务 `priority = P2` 且 `complexity_score ≤ 3`
3. 任务无 `blocked_by` 依赖
4. 同时满足 → 跳过 dispatch queue, Agent 直接 `acceptTask + commit + spawn-result`

极速模式不引入新 API, 只是 coordinator 内部 fast path.

### 3.5 与 V1.0 (multi-agent-dev-skill) 关系

V1.0 是 Claude Code subagent 队列架构 (queue.json + agent-runner.sh), 适合 **代码批量生成** 场景 (2026-04 17 模块开发). V5.0 是 **OpenClaw 原生 Agent 部门协同**, 适合 **跨部门业务 dispatch** 场景.

两者并存:
- 新业务 dispatch 任务 → V5.0 部门 Agent
- 代码生成/批量重构任务 → V1.0 multi-agent-dev-skill (保留, 不并入)

---

## 四、验证方法

### 4.1 Phase 1 验证 (本次提交)

| 检查项 | 命令/方法 | 期望 |
|--------|----------|------|
| openclaw.json 合法 JSON | `python3 -m json.tool /root/.openclaw/openclaw.json` | 无报错 |
| agents.list 含 5 项 | `jq '.agents.list \| length' openclaw.json` | `5` |
| 3 个 workspace 存在 | `ls -d /root/.openclaw/workspace/agents/{xs,cc,cw}-agent` | 3 个目录 |
| 每个 workspace 有 SOUL.md | `for d in xs cc cw; do test -f agents/$d-agent/SOUL.md; done` | exit 0 |
| 每个 workspace 有 AGENTS.md | 同上 | exit 0 |
| 部门 Agent 不在 acp.allowedAgents | `jq '.acp.allowedAgents // [] \| map(select(. == "xs-agent")) \| length' openclaw.json` | `0` (允许缺失) |

### 4.2 Phase 2 验证 (后续)
- `department_knowledge` 表创建成功 (psql `\d`)
- POST `/api/v1/dev/knowledge` 401→201→GET 200
- Agent 启动时自动 load 自己部门的 knowledge

### 4.3 Phase 3 验证 (后续)
- P2 单部门任务 dispatch metrics 中 `fast_path=true` 占比 ≥ 40%
- fast path 任务 P95 完成时间 ≤ 60s

### 4.4 端到端验证
任务派发跑通场景: 销售部 (xs-agent) 收到订单 dispatch → sessions_send 转仓配部 (cw-agent) → cw 完成 → sessions_send 回 xs → xs commit + spawn-result 上报.

---

## 五、变更清单 (本任务实际提交)

```
M  /root/.openclaw/openclaw.json                              # agents.list 扩展 (2→5)
A  docs/MULTI-AGENT-DESIGN-V5.md                              # 本文档
A  agents/xs-agent/SOUL.md                                    # 销售部
A  agents/xs-agent/AGENTS.md                                  # 销售部
A  agents/xs-agent/CLAUDE.md                                  # 销售部
A  agents/cc-agent/SOUL.md                                    # 客服部
A  agents/cc-agent/AGENTS.md                                  # 客服部
A  agents/cc-agent/CLAUDE.md                                  # 客服部
A  agents/cw-agent/SOUL.md                                    # 仓配部
A  agents/cw-agent/AGENTS.md                                  # 仓配部
A  agents/cw-agent/CLAUDE.md                                  # 仓配部
```

注: `before-dispatch.js` hook 落到 coordinator 实际接入留待后续任务 (需先确认 coordinator 启动方式, 避免在 V5.0 Phase 1 抢跑造成 misroute).

---

## 六、参考

- V1.0 架构: `agents/multi-agent-dev-skill/MULTI-AGENT-ARCHITECTURE.md` (Claude Code subagent 队列, 历史档案)
- V3.1 dispatcher: `coordinator/devlog-pg-listen.service` (NOTIFY 事件驱动)
- sessions_send 在 OpenClaw 原生 Agent 间工作: 任务 issues (V4.0 废弃原因不成立, 2026-06-15 海刚复核)
- 任务记录: `T2026-0509-010` in `/api/v1/dev/tasks/`