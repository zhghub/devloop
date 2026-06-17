# DevLoop V2.3 系统方案 (MASTER 终稿)

> **文档定位**: 本文档是 DevLoop V2.3 系统的**唯一权威方案**,取代旧的 DEVLOG-MASTER-PLAN / DEVLOG-OPTIMIZATION-PLAN / DEVLOG-FEATURES / DEVLOG-AUTONOMOUS-CHAIN / DEVLOG-PRODUCT-MANUAL / 小辉数字员工2.0-DevLoop开发中枢系统说明 / V23-DEPLOYMENT-REPORT / V23-DELETION-LOG 共 8 份旧文档,以及 docs/legacy-plans/ 17 个 5/22 之前的旧方案
> **版本**: V2.3 (终稿, 2026-06-10)
> **最后更新**: 2026-06-10 16:58 GMT+8
> **适用范围**: 任何 OpenClaw 实例迁过来都按本文档对齐

---

## 第一章: 系统定位

### 1.1 核心定义

**DevLoop = 小辉数字员工2.0的 AI 开发任务调度中枢**。负责: 任务管理、多 Agent 调度、代码审计、准则注册、仪表盘、通知。

### 1.2 范式 (永久铁律)

```
┌────────────────────────────────────────────────────┐
│ DevLoop = 数据底座 (Fastify+Prisma+PostgreSQL)       │
│   - 持久化 + 检索, 不做决策                         │
│   - 12 张表 + 80+ API + 4 层硬约束                  │
│   - 不靠 MEMORY.md 记忆, 迁到任何 OpenClaw 都能跑    │
└────────────────────────────────────────────────────┘
                          ↕ 调 API
┌────────────────────────────────────────────────────┐
│ OpenClaw 大脑 + ACP Agent = 智能体层                │
│   - openclaw-main: 调度中枢 + 决策                  │
│   - claude-main: 主力开发 (代码/bug/refactor/test)  │
│   - codex: 代码审查 (review/security/architecture)  │
│   - deepseek: 深度审计 (audit/security-scan)         │
└────────────────────────────────────────────────────┘
```

### 1.3 历史演进 (V1.0 → V2.3)

| 版本 | 时间 | 核心 | 状态 |
|---|---|---|---|
| V1.0 | 2026-04-25 | 单脚本 (completeness-cron) | 已废弃 |
| V2.0 | 2026-05-12 | ai-audit-engine + 智能分发 | 已废弃 |
| V3.0 | 2026-06-02 | AI 原生架构定稿, 14 表 | 文档已废,代码沿用 |
| V4.0/V4.1 | 2026-06-03 | 9 环闭合, 5 引擎 | 文档已废,代码沿用 |
| **V2.3** | **2026-06-09 14:23** | **openclaw 大脑 + 真活 agent + DB 队列** | **当前生产** |

### 1.4 当前状态 (2026-06-10 16:58 实际)

- ✅ **真活 agent**: 2 个 (openclaw + claude-main)
- ❌ **codex + deepseek CLI**: 装在宿主机但容器内 candidates 找不到 (待修, Stage 6)
- ⚠️ **21 个 pending 任务** (P1: 16, P2: 5) 卡住
- ⚠️ **V5 scheduler** enabled+active+Restart=always (PID 3813 14:26 复活, 抢任务)
- ⚠️ **cron 677d03ce** "DevLoop自动修复循环" 99 次 skipped, 根本没调度
- ✅ **4 层硬约束**: 全绿 (api/db/publish/consumer)
- ⚠️ **dev_spawn_queue** 主路径走 DB (V23), 文件路径 fallback 仍叫 .v5-spawn-queue.jsonl (历史包袱, 不动)

---

## 第二章: 4 角色分工 (永久铁律)

### 2.1 角色定义

| 角色 | agent_name | CLI | caps | 职责 |
|---|---|---|---|---|
| **leader** | openclaw | (self) | task_scheduling, audit, notification, architecture, design, planning, docs | 调度中枢, 不领任务, 不写代码 |
| **developer** | claude-main | claude | code, feature, bug, refactor, test | 主力写手, 编码/Bug 修复/重构/测试 |
| **reviewer** | codex | codex | review, security, architecture | 代码审查, 安全审计, 架构验证 |
| **auditor** | deepseek | deepseek | audit, security-scan, deep-analysis | 深度分析, 全栈审计, 代码扫描 |

### 2.2 任务类型映射 (matchTeammate)

```js
const map = {
  'plan':         'leader',     // 方案/规划/架构设计 (Stage 3 新增)
  'feature':      'developer',
  'bug':          'developer',
  'refactor':     'developer',
  'security':     'auditor',    // 改 auditor, security-scan 匹配
  'architecture': 'developer',  // 改 developer, 不走 openclaw
  'review':       'auditor',    // 改 auditor
  'audit':        'auditor',
  'test':         'developer',
  'docs':         'developer',  // 改 developer, openclaw 也不适合写文档
  'migrate':      'developer',
  'default':      'developer'
};
```

### 2.3 真活检测铁律 (2026-06-07 海刚 03:08 铁律 #49)

> "在职" = DB online AND 主机 CLI 真装了

- **agent-registry-truthy.probeAgent(cli)**: 实测 CLI `--version` 可执行
- **TRUTHY_ROLES 白名单**: 不在白名单的历史 agent (cbc/gemini-designer/...) → 强制 offline
- **真活 agent 池** = 白名单 ∩ (DB online AND 10min 内有心跳)
- **降级**: 主 agent offline → 调 smartMatchAgent 兜底
- **不可用兜底都没** → 任务跳过 (skipped_unavailable)

### 2.4 永远不动的 (openclaw 铁律)

- openclaw = 调度中枢, **不领任务、不写代码**
- 4 角色架构不能少 (leader/developer/reviewer/auditor)
- TRUTHY_ROLES 白名单不能加新 agent 不测试就注册

---

## 第三章: 完整数据流 (V2.3)

### 3.1 主路径 (openclaw 大脑直接调度, V2.3 范式)

```
openclaw-main (cron 677d03ce, 30m tick)
    │
    ├─ Step 1: 验证 4 层硬约束
    │    GET /api/v1/dev/context/contract/status
    │    不绿 → 飞书群告警 + 退出
    │
    ├─ Step 2: 查 pending 任务
    │    GET /api/v1/dev/tasks?status=pending&limit=10
    │    空 → HEARTBEAT_OK 退出
    │
    ├─ Step 3: 4 段结构校验 (Stage 4 改造)
    │    grep "## 问题 / ## 目标 / ## 方案 / ## 验证"
    │    缺段 → 写 issues + 飞书群通知
    │
    ├─ Step 4: 调 team-dispatch 派发
    │    POST /api/v1/dev/worker/team-dispatch {limit:5}
    │
    ├─ Step 5: 直接 acpx exec 派活 (Stage 6 完成后)
    │    acpx exec --file /tmp/prompt-{task_id}.md
    │    agent_name=claude-main/codex/deepseek
    │
    └─ Step 6: 完成后回写
         POST /api/v1/dev/worker/spawn-result
         body: {task_id, agent_name, exit_code, commit_hash}
```

### 3.2 任务状态机

```
pending ──派发──→ in_progress ──完成──→ completed
   ↑                  │                      │
   │                  │ (30min 无响应)       │ (commit_hash 为空)
   │                  ↓                      ↓
   └─ reset-orphans  回 pending        issues 写 + 重派
   │
   └─ discarded (V3.5 FP 拦截, 字面假阳性)
```

### 3.3 任务派发算法 (team-dispatcher.js teamDispatch)

```js
async function teamDispatch(taskTree) {
  // 1. 真活 agent 池 (DB + 真探活双校验)
  availableAgents = await truthy.getAvailableAgents();

  // 2. 任务排序: P0→P1→P2, 同模块聚类, created_at ASC
  sorted = ...preCheckAndDiscard + sort

  // 3. 拆分 leaderTasks (含"方案/规划") vs teammateTasks
  leaderTasks = sorted.filter(t => 
    t.title?.match(/方案|规划|架构设计|planning/i)  // Stage 3 改
  );

  // 4. leaderTasks 最多 1 个 → openclaw
  for (const task of leaderTasks.slice(0, 1)) {
    // 派给 openclaw (leader)
    // 2h 工作锁
  }

  // 5. teammateTasks 最多 10 个 → matchTeammate 匹配 → enqueueSpawn
  for (const task of teammateTasks.slice(0, 10)) {
    const role = matchTeammate(inferTaskType(task));
    // 降级: role 不可用 → smartMatchAgent 兜底
    // 兜底都失败 → 跳过
  }
}
```

### 3.4 关键路径 (旧 V5 已废, V2.3 走 DB)

| 旧 V5 路径 | 新 V2.3 路径 | 状态 |
|---|---|---|
| `.v5-spawn-queue.jsonl` (文件) | PG `dev_spawn_queue` 表 | ✅ V2.3 主路径 |
| `spawn-consumer-v6.py` (rc=0 假死) | `spawn-consumer-v23.py` (DB 真活) | ✅ V2.3 |
| `v5-scheduler.service` (systemd 抢活) | **删, openclaw cron 接管** | ✅ **已杀 (6/11 25fe442d)** |
| `v5-scheduler.py` (CLI 派发) | **删** | ✅ **已杀 (6/11 25fe442d)** |
| `scheduler-v23.py` (过渡总指挥) | **删, openclaw 直调** | ✅ **已杀 (6/11 25fe442d)** |
| `spawn-consumer-v23.sh` (过渡 wrapper) | **删, openclaw 直接 acpx exec** | ✅ **已杀 (6/11 25fe442d)** |
| `spawn-consumer-v23.py` (V23 worker) | **删, V3.1 daemon 接管** | ✅ **已杀 (6/15 17:05 海刚 STOPPED-0615-1557)** |
| `devlog-team-dispatch-cron.sh` (机械拉 cron) | **删, V3.1 NOTIFY 事件驱动** | ✅ **已杀 (6/15 17:05)** |
| `minimax-heartbeat.sh` (机械 1min 拉) | **删, V3.1 大脑决策** | ✅ **已杀 (6/15 17:05)** |
| `devlog-auto-dispatch.sh` (V2.0 V3.1 修订文档 #10 标死) | **删, V3.1 daemon 接管** | ✅ **已杀 (6/15 17:22)** |
| `backend/src/modules/devlog/scripts/devlog.sh` (9299 字节) | **删, AGENTS.md 用根级 `scripts/devlog.sh`** | ✅ **已杀 (6/15 17:22)** |
| `scripts/spawn-consumer-s1-check.sh` (V23 配套) | **删** | ✅ **已杀 (6/15 17:05)** |

> **2026-06-15 Stage 2 + Stage 7 全部清零** (海刚 16:55 强制, 6/15 17:05+17:22 双轮清理)
> V3.1 范式下唯一 dispatcher: `/etc/systemd/system/devlog-pg-listen.service` (宿主机) + `scripts/devlog-pg-listen-host.js` (300 行)

---

## 第四章: API 端点总览 (80+ 端点)

> 全部端点都在 `/api/v1/dev/` 前缀下, **全部** 需要 `x-internal-secret` header (除 `/api/v1/auth/*` 公开端点)

### 4.1 核心调度

| 端点 | 方法 | 说明 |
|---|---|---|
| `/worker/team-dispatch` | POST | 🎯 V2.3 主入口, openclaw 30m tick 调 |
| `/worker/dispatch-status` | GET | agent + 队列 + 任务统计 |
| `/worker/queue-status` | GET | 队列状态 |
| `/worker/enqueue` | POST | 手动入队 |
| `/worker/spawn-queue` | GET | 查看队列 |
| `/worker/spawn-ack` | POST | 确认任务已 spawn |
| `/worker/spawn-result` | POST | 接受结果 → 更新 task 状态 |

### 4.2 Agent 管理

| 端点 | 方法 | 说明 |
|---|---|---|
| `/worker/agents/health` | GET | 全部 agent 健康 |
| `/worker/agents/heartbeat` | POST | 单 agent 心跳 |
| `/worker/agents/detect` | POST | CLI 探活 + 同步 DB |
| `/worker/agents/reset-orphans` | POST | 清孤儿 in_progress (30min) |
| `/worker/agents/activity` | GET | 活动日志 |
| `/agents` | GET | agent 注册列表 |
| `/agents/:name/pause\|resume\|stop\|restart\|kick` | POST | agent 控制 |
| `/agents/:name/reassign-tasks` | POST | 重新分配 |
| `/agents/register` | POST | 注册新 agent |

### 4.3 任务 CRUD

| 端点 | 方法 | 说明 |
|---|---|---|
| `/tasks` | GET/POST | 列表/创建 (4 层硬约束) |
| `/tasks/:id` | GET/PATCH/DELETE | 查/改/删 |
| `/logs` | GET | 同 `/tasks` 别名 |
| `/summary` | GET | 按模块汇总 |
| `/stats` | GET | 数字统计 |
| `/dashboard` | GET | 仪表盘 |

### 4.4 准则 & 经验

| 端点 | 方法 | 说明 |
|---|---|---|
| `/guidelines` | GET/POST | 铁律准则 |
| `/guidelines/:id` | PUT/DELETE | 改/删 |
| `/lessons` | GET/POST | 经验教训 |
| `/lessons/:id` | PUT/DELETE | 改/删 |

### 4.5 审计 & 审查

| 端点 | 方法 | 说明 |
|---|---|---|
| `/project/review` | POST | 代码审查 |
| `/project/review-status` | GET | 审查进度 |
| `/project/review/pause\|resume` | POST | 暂停/恢复 |
| `/audit/run` | POST | 跑审计 |
| `/audit/preview` | POST | 审计预览 |
| `/quality-checks` | GET/POST | 质量检查 |
| `/context/contract/status` | GET | 🎯 4 层硬约束自检 |
| `/module-tasks/generate` | POST | 补任务 (publish-completeness-tasks.py 调) |

### 4.6 仪表盘

| 端点 | 方法 | 说明 |
|---|---|---|
| `/dashboard/radar` | GET | 雷达图 |
| `/dashboard/health-score` | GET | 健康度 |
| `/dashboard/module-completion` | GET | 模块完成度 |
| `/dashboard/overview` | GET | 总览 |
| `/dashboard/at-risk` | GET | 风险模块 |
| `/workboard` | GET | 工作台总览 |
| `/workboard/agents` | GET | agent 列表 |
| `/workboard/tasks` | GET | 任务看板 |
| `/workboard/health` | GET | 健康度 |
| `/workboard/openclaw` | GET | OpenClaw 状态 |
| `/workboard/v5-scheduler` | GET | V5 状态 (Stage 2 后会变 stale) |

### 4.7 通知 & 公告

| 端点 | 方法 | 说明 |
|---|---|---|
| `/notify/audit-summary` | POST | 审计摘要通知 |
| `/notify/critical-alert` | POST | 严重告警 |
| `/announcements` | GET/POST | 公告 CRUD |
| `/announcements/:id/read` | POST | 标记已读 |
| `/announcements/:id/archive` | POST | 归档 |

---

## 第五章: 12 张数据库表

| # | 表名 | 列数 | 用途 | 关键列 |
|---|---|---|---|---|
| 1 | **dev_task_logs** | 31 | 🎯 任务主表 | task_id, module, title, status, priority, agent_name, commit_hash, files_changed, issues, solutions, learnings |
| 2 | **dev_agent_registry** | 19 | Agent 注册表 | agent_name, status, capabilities, last_heartbeat_at, max_concurrent |
| 3 | **dev_agent_activity** | 8 | Agent 活动日志 | agent_name, action, target_type, details |
| 4 | **dev_guidelines** | 8 | 铁律/准则 | category, title, content, priority |
| 5 | **dev_lessons_learned** | 16 | 经验教训 | task_id, lesson_type, content, severity |
| 6 | **dev_announcements** | 10 | 公告 | title, content, severity, target_agents |
| 7 | **dev_work_locks** | 9 | 文件锁 | target_type, target_path, locked_by, expires_at |
| 8 | **dev_decisions** | 12 | 决策记录 | title, context, decision, alternatives |
| 9 | **dev_quality_checks** | 11 | 质量检查 | module, check_type, score, findings |
| 10 | **dev_config_history** | 11 | 配置变更史 | config_key, old_value, new_value |
| 11 | **dev_resource_stats** | 10 | 资源统计 | agent_name, duration_seconds, avg_cpu_percent, peak_memory_mb |
| 12 | **dev_module_registry** | 7 | 模块注册 | module_name, completion_pct, status |

### 5.1 关键 SQL

```sql
-- 获取在线 agent
SELECT agent_name, status, last_heartbeat_at FROM dev_agent_registry
WHERE status IN ('online','busy') AND last_heartbeat_at > NOW() - INTERVAL '10 minutes';

-- 拉取待处理任务 (P0→P1→P2, 同模块聚类)
SELECT * FROM dev_task_logs WHERE status='pending'
ORDER BY CASE priority WHEN 'P0' THEN 0 WHEN 'P1' THEN 1 ELSE 2 END, module ASC, created_at ASC LIMIT 8;

-- 虚假完成任务检测
SELECT * FROM dev_task_logs WHERE status='completed' AND commit_hash IS NULL;

-- 孤儿 in_progress 重置
UPDATE dev_task_logs SET status='pending', agent_name=NULL
WHERE status='in_progress' AND updated_at < NOW() - INTERVAL '30 minutes';

-- 4 层硬约束查询
SELECT verifyDevLoopContract() AS status;

-- dev_spawn_queue 队列 (V2.3 主路径)
SELECT * FROM dev_spawn_queue WHERE spawned_at IS NULL AND completed_at IS NULL
ORDER BY CASE priority WHEN 'P0' THEN 0 WHEN 'P1' THEN 1 WHEN 'P2' THEN 2 ELSE 3 END, enqueued_at ASC
LIMIT 20;
```

---

## 第六章: 4 层硬约束 (2026-06-10 海刚强制永久铁律)

> **任何 OpenClaw 实例迁过来都必须保证这 4 层机制就位**。`GET /api/v1/dev/context/contract/status` 返 `data.ok=true` 才算通过

### 6.1 第 1 层: API 拦截 (tasks.js L138-146)

```js
const _desc = (description || '').trim();
if (_desc.length > 0 && _desc.length < 50) {
  return reply.code(400).send({
    success: false,
    error: `description 详细度不足: ${_desc.length}/50 字符`,
    hint: '必须包含 ## 问题 / ## 目标 / ## 修复方案 / ## 验证方法 四个章节'
  });
}
```

### 6.2 第 2 层: Publish 自动拼装 (publish-completeness-tasks.py)

```python
# post_task 调用时自动从 goal+approach 拼装 4 段式 description
description = f"""## 目标
{goal}

## 修复方案
{approach}
"""
```

### 6.3 第 3 层: Consumer Fallback (spawn-consumer-v23.py L184-200)

```python
# description 为空 + goal/approach 非空 → fallback 拼装
if not desc and (goal or approach):
    desc = f"## 目标\n{goal}\n\n## 修复方案\n{approach}"
```

### 6.4 第 4 层: DB Schema 守护 (devlog-contract-guard.js)

```js
// 启动时验证 dev_task_logs.description 列存在 + type=text
// 失败 → 后端拒绝启动 + 飞书告警
```

### 6.5 监控指标

| 指标 | 当前 | 目标 |
|---|---|---|
| good_desc 比例 (24h) | 79.1% | ≥ 70% ✅ |
| 任务平均处理时间 | ~10min | < 5min |
| API 400 拒绝率 (新任务) | 监控 | >5% 报警 |

### 6.6 4 段格式铁律 (任何 dispatcher 必填)

```
## 问题
哪个文件/行号/症状 ("attendance/records 返回 500")

## 目标
修复后要达到什么 ("7 个前端页面全可调用后端")

## 修复方案
具体步骤 ("创建 routes/attendance.js + 引用 3 个 services + 暴露 31 端点")

## 验证方法
怎么确认修好了 ("curl GET /api/v1/attendance/records → 200")
```

---

## 第七章: 部署架构 (V2.3 端到端)

### 7.1 部署时间

- **2026-06-09 14:23**: scheduler-v23.py 部署
- **2026-06-09 14:42**: V2.3 SMART-DISPATCH API 部署
- **2026-06-09 15:50**: spawn-consumer-v23.py + .sh 部署
- **2026-06-09 19:20**: 修复 V6 死链 (改读 DB)
- **2026-06-10 04:02**: 4 层硬约束上线
- **2026-06-10 16:58**: MASTER 文档重组 (本文件)

### 7.2 关键组件

```
后端 (Docker 容器 xiaohui-2-backend):
  ├── src/modules/devlog/  45 文件
  │   ├── routes/  18 路由文件
  │   ├── services/  11 服务文件
  │   └── _split/  30+ 子路由
  └── Fastify + Prisma + PostgreSQL

host:
  ├── scripts/
  │   ├── agent-heartbeat.sh   # 心跳 (30s)
  │   ├── pg-dump-daily.sh     # DB 备份 (03:00)
  │   ├── disk-monitor.sh      # 磁盘监控 (15min)
  │   ├── pg-slow-query-check.sh  # 慢查询审计 (周一 09:00)
  │   ├── audit-fake-complete.sh  # 假完成审计 (6h)
  │   ├── reap-stale-spawn-queue.sh  # 队列回收 (15min)
  │   ├── lint-cron.sh         # lint 巡检 (6h)
  │   ├── verify-devlog-contract.sh  # 4 层硬约束验证
  │   └── publish-completeness-tasks.py  # 补任务
  # ↑ 以下 7 个文件 6/11 25fe442d + 6/15 17:05+17:22 全部 git rm
  #   spawn-consumer-v23.sh / spawn-consumer-v23.py / scheduler-v23.py
  #   v5-scheduler.py / v5-scheduler-wrapper.py
  #   devlog-team-dispatch-cron.sh / minimax-heartbeat.sh / devlog-auto-dispatch.sh
  # 取代方案: V3.1 宿主机 systemd `devlog-pg-listen.service` (300 行, 真活)
  ├── systemd:
  │   └── (无 — V3.1 不需要 systemd service 起 devlog worker, 唯一 service 是 `devlog-pg-listen.service` 宿主机跑)
  └── OpenClaw cron:
      ├── 677d03ce DevLoop自动修复循环  # ⏳ 改写 (Stage 5)
      ├── 57330452 session-context-manager (30m)
      ├── 63fa1f81 Module Completeness Audit (0 */6h)
      ├── b879cd4a DevLoop 4层硬约束自检 (0 */6h)  # ✅ 真活
      ├── 6f79b68c DevLoop全系统自检 (3h)
      ├── 70470ccb fullstack-health-audit (4h)
      ├── 6cccfcce code-quality-scanner (8h)
      ├── 74e8f777 每日开发日报 (0 9 * * 1-5)
      ├── fbccc19c Memory Dreaming Promotion (0 3 * * *)
      └── 193db4d4 workbuddy-5day-review (6/12 02:00)
```

### 7.3 5 旧脚本废弃清单 (2026-06-09 14:23 海刚 + openclaw-main)

1. `completeness-cron.sh` → `.disabled.v23/`
2. `agent-heartbeat-v2.sh` → `.disabled.v23/`
3. `audit-publish-tasks.py` → `.disabled.v23/`
4. `module-http-probe.py` → `.disabled.v23/`
5. `module-completeness-audit.py` → `.disabled.v23/`

**备份位置**: `/home/workspace/.deprecated-scripts/2026-06-09-v23-migration/`

### 7.4 V2.3 5 永久铁律

1. **旧方案隔离**: mv .trash.v23/ → 24h 缓冲 → 验证系统正常 → rm
2. **容器内服务探活**: 不依赖 sql UPDATE 假数据, 真探活 (acpx --version)
3. **队列路径对齐**: DB 主路径 (`dev_spawn_queue` 表), 文件 fallback
4. **V2.3 路由补全**: `POST /api/v1/dev/worker/team-dispatch` 替代 V5 旧派发
5. **真活 agent 池**: TRUTHY_ROLES 白名单 + DB online + 10min 心跳

### 7.5 调度路径变更

| 旧 V5 (待删) | 新 V2.3 (当前) |
|---|---|
| crontab `*/1 * * * *` 调 v5-scheduler.py | openclaw cron 677d03ce 30m tick (Stage 5 改写) |
| v5-scheduler.service systemd 守护 | (V2.3 不需要 service, openclaw 自带心跳) |
| 队列文件 `.v5-spawn-queue.jsonl` | DB 表 `dev_spawn_queue` |
| 派发: acpx exec (--file prompt) | openclaw 直接 acpx exec (Stage 5 实现) |
| 5 个 .disabled.v23 旧脚本 | 删除 (Stage 2+7) |

---

## 第八章: 任务格式与铁律

### 8.1 任务必填字段 (4 段铁律)

```yaml
module: "模块名"
priority: "P0|P1|P2"
title: "简短描述 (≤ 100 字符)"
description: |
  ## 问题
  哪个文件/行号/症状
  
  ## 目标
  修复/开发后要达到什么
  
  ## 修复方案
  具体步骤
  
  ## 验证方法
  怎么确认
agent_name: "openclaw-main"  # 默认发布者
agent_type: "code|system-architect|ops|qa"
```

### 8.2 任务发布铁律 (2026-06-10 海刚永久)

> **任务的发布一定是要详细任务不能只有一个标题,让其他 agent 凭一个标题就动手,结果完成任务跑题和打不到预期要求**

### 8.3 dispatcher 脚本清单 (必填 description)

| 脚本 | 路径 | 调用方式 |
|---|---|---|
| publish-completeness-tasks.py | scripts/ | post_task API (已自动拼装) |
| publish-fp-fixes.py | scripts/ | post_task API (已自动拼装) |
| dispatch-team.py | scripts/ | post_task API (已自动拼装) |
| publish-task.py | scripts/ | post_task API (已自动拼装) |
| claude-code-dispatcher | scripts/ | post_task API (已自动拼装) |

### 8.4 V3.5 False-Positive Patterns (创建时拦截)

```js
const FP_CREATION_PATTERNS = [
  [/空数据|data.*为空|返回.*空|data=\[\]/i, 'auth_required'],
  [/\/api\/v1\/erp\/(dashboard|products|...)/i, 'erp_prefix'],
  [/P\d.*need.*detailed audit|module.*need.*audit/i, 'meta_audit'],
  [/Module.*completion.*review|模块.*完成度/i, 'meta_review'],
  [/全栈健康审计.*通过|全栈健康审计.*clear/i, 'meta_health'],
];
```

**28 个字面假阳性任务已被 V3.5 拦截闭环** (T20260610-14662/14676/14682/14683/14685/14686/14687/14689/14690/14691/14692/14693 等)

---

## 第九章: 回滚方案

### 9.1 隔离目录 (24h 缓冲)

```
/home/ubuntu/workspace/.trash.v23/
├── docs/                         # Stage 1 隔离
│   ├── DEVLOG-MASTER-PLAN.md
│   ├── DEVLOG-OPTIMIZATION-PLAN.md
│   ├── DEVLOG-FEATURES.md
│   ├── DEVLOG-AUTONOMOUS-CHAIN.md
│   ├── DEVLOG-PRODUCT-MANUAL.md
│   ├── 小辉数字员工2.0-DevLoop开发中枢系统说明.md
│   ├── V23-DEPLOYMENT-REPORT.md
│   ├── V23-DELETION-LOG.md
│   └── legacy-plans/             # 17 个 5/22 之前的旧方案
├── etc-systemd/                  # Stage 2 隔离
│   └── v5-scheduler.service
├── scripts/                      # Stage 2+7 隔离
│   ├── v5-scheduler.py
│   ├── v5-scheduler-wrapper.py
│   ├── scheduler-v23.py
│   ├── spawn-consumer-v23.py
│   └── spawn-consumer-v23.sh
├── host/                         # Stage 2 隔离
│   ├── .v5-heartbeat.json
│   ├── .v5-scheduler/
│   ├── .v5-prompts/
│   └── .v5-runner.sh
├── STAGE1-CHECKPOINT.md
├── STAGE2-SOP.md
├── STAGE3-PATCH.md
├── STAGE5-CRON-PATCH.md
├── STAGE6-PLAN.md
└── STAGE1-DONE.md
```

### 9.2 回滚操作 (24h 内)

1. **文档回滚**: `mv .trash.v23/docs/*.md → /root/.openclaw/workspace/agents/xiaohui-2.0-dev/docs/`
2. **脚本回滚**: `mv .trash.v23/scripts/v5-scheduler.* → /home/ubuntu/workspace/scripts/`
3. **systemd 回滚**: `mv .trash.v23/etc-systemd/v5-scheduler.service → /etc/systemd/system/`
4. **systemctl daemon-reload && systemctl enable v5-scheduler && systemctl start v5-scheduler**
5. **任务池重置**: `UPDATE dev_spawn_queue SET spawned_at=NULL, completed_at=NULL WHERE ...`
6. **git revert** 对应 commit

### 9.3 不可回滚 (审计资产保留)

- `dev_spawn_queue` 表 (V5 任务历史, 审计需要)
- `dev_task_logs` 表 (所有任务历史)
- `dev_agent_registry` 表 (所有 agent 状态历史)
- `dev_announcements` 表 (所有公告历史)

### 9.4 8 Stage 进度追踪 (2026-06-10 16:58)

| Stage | Task ID | 状态 | 备注 |
|---|---|---|---|
| 1 文档重组 | T20260610-14737 | in_progress | 写本文件 + 重写 MAP |
| 2 杀 V5 | T20260610-14738 | in_progress | systemd 杀 + 脚本隔离 |
| 3 方案专家 | T20260610-14739 | pending | matchTeammate 加 plan 路径 |
| 4 4 段校验 | T20260610-14740 | pending | openclaw 派发层强制 4 段 |
| 5 cron 改造 | T20260610-14741 | in_progress | 677d03ce text 重写 |
| 6 agent 全员 | T20260610-14744 | pending | 容器 candidates 加 /host-nvm |
| 7 删 V23 | T20260610-14742 | pending | 删过渡脚本 |
| 8 e2e 测试 | T20260610-14743 | pending | 写 devlog-v23-e2e-test.sh |

---

## 附录 A: 旧版本索引 (历史归档)

| 文档 | 行数 | V 版本 | 归档路径 |
|---|---|---|---|
| DEVLOG-MASTER-PLAN.md | 290 | V3.0 | .trash.v23/docs/ |
| DEVLOG-OPTIMIZATION-PLAN.md | 492 | V3.0 | .trash.v23/docs/ |
| DEVLOG-FEATURES.md | 114 | V3.0 | .trash.v23/docs/ |
| DEVLOG-AUTONOMOUS-CHAIN.md | 507 | V4.0/V4.1 | .trash.v23/docs/ |
| DEVLOG-PRODUCT-MANUAL.md | 515 | V4.0 | .trash.v23/docs/ |
| 小辉数字员工2.0-DevLoop开发中枢系统说明.md | 857 | V4.1 | .trash.v23/docs/ |
| V23-DEPLOYMENT-REPORT.md | 135 | V2.3 部署 | .trash.v23/docs/ |
| V23-DELETION-LOG.md | 113 | V2.3 删除 | .trash.v23/docs/ |
| legacy-plans/*.md | 17 文件 | V1.0-V4.0 | .trash.v23/docs/legacy-plans/ |
| DEVLOG-MAP.md | 473 → 重写 | V2.3 源码地图 | docs/DEVLOG-MAP.md (新版) |

## 附录 B: 永久铁律清单

1. **V2.3 范式铁律**: openclaw 大脑直接调度, 不用脚本 (Stage 5 落地)
2. **任务 4 段铁律**: ## 问题 / ## 目标 / ## 修复方案 / ## 验证方法 (4 层硬约束保底)
3. **真活 agent 铁律**: TRUTHY_ROLES 白名单 + DB online + 10min 心跳
4. **4 角色铁律**: leader/developer/reviewer/auditor 不能少
5. **源码保护铁律**: 24h 缓冲 + .trash.v23/ 隔离, 不直接 rm
6. **dispatcher 4 段铁律**: post_task 必须带 description 4 段, 缺段被 V3.5 拦截
7. **任务发布铁律**: 详细任务 (4 段), 不是只发标题
8. **代码改动铁律**: 动手前完整读源码, 不靠记忆 (海刚 16:29 强化)
9. **公告事实铁律**: 任何 agent 发公告前必须 md5 + ls -la 双重验证
10. **回滚铁律**: 24h 隔离期内可以一键回滚, 过期后用 git revert

---

## 第十章: V2.3 8 Stage 完成总结 (2026-06-10 17:33)

### 10.1 时间线

```
2026-06-10 14:23 - 18:08: 海刚拉齐 8 Stage 计划
2026-06-10 16:35: Stage 5 (cron 改造) 抢跑 d6d136cb
2026-06-10 16:39: Stage 6 (agent 全员) 抢跑 e9ffff2b
2026-06-10 16:50: Stage 7 (删 V23 脚本) 抢跑 6cbde8d0
2026-06-10 17:00: Stage 5 cron 真正落地 d6d136cb (complete)
2026-06-10 17:05: Stage 8 (e2e 修字段路径) 抢跑 a5be96a7
2026-06-10 17:08: Stage 4 (4 段校验) 抢跑 245b1ce1
2026-06-10 17:30: Stage 2 (杀 V5) 隔离完成
2026-06-10 17:32: Stage 1 (文档重组) commit 2637e9b9
2026-06-10 17:33: Stage 3 (方案专家) commit 0068257b
```

### 10.2 8 Stage 完成矩阵

| Stage | Task | 我做的 | 抢跑 | 最终 commit |
|---|---|---|---|---|
| 1 文档重组 | T20260610-14737 | ✅ 2637e9b9 | - | 2637e9b9 |
| 2 杀 V5 | T20260610-14738 | ✅ 隔离完成 | - | (无 git, 物理删除) |
| 3 方案专家 | T20260610-14739 | ✅ 0068257b | - | 0068257b |
| 4 4 段校验 | T20260610-14740 | - | ✅ 245b1ce1 | 245b1ce1 |
| 5 cron 改造 | T20260610-14741 | - | ✅ d6d136cb | d6d136cb |
| 6 agent 全员 | T20260610-14744 | - | ✅ e9ffff2b (DB 完整) | e9ffff2b |
| 7 删 V23 脚本 | T20260610-14742 | - | ✅ 6cbde8d0 | 6cbde8d0 |
| 8 e2e 测试 | T20260610-14743 | - | ✅ a5be96a7 | a5be96a7 |

### 10.3 待海刚决策 2 项

1. **Stage 6 容器残留** (DB 假活):
   - 选项 1: docker-compose mount 宿主机 /root/.nvm → /host-nvm + candidates 加 /host-nvm (推荐)
   - 选项 2: 容器内 npm install (codex 需 ChatGPT OAuth)
   - 选项 3: 接受现状 (DB 假活, cron 真调会失败)

2. **后端重启** (Stage 3 改动):
   - docker compose restart xiaohui-2-backend && sleep 18
   - 不动 docker 网络, 应该安全

---

## 第十一章: V2.3 大脑调度范式 (海刚 2026-06-10 21:22 拍板, 永久)

### 11.1 范式铁律

> **调度 = openclaw 大脑主动调 API, 不是脚本定时**

**之前的错误理解**:
- ❌ 用 crontab shell 脚本定时调 spawn-consumer-v23.py
- ❌ 让 Python 脚本自己定时跑
- ❌ `*/1 * * * * python3 dispatch.py` 这种 V5 风格

**V2.3 正确范式**:
- ✅ OpenClaw 内部 cron (agentTurn payload) → 唤起 openclaw 大脑
- ✅ 大脑主动分析 → 调 team-dispatch API → 派任务
- ✅ 大脑主动监控 → 处理孤儿 → 输出决策报告
- ✅ 调度逻辑在 openclaw 大脑里, 不在 shell 脚本里

### 11.2 实施: cron job `1028be6f-d6cb-4d5b-bcbb-cf7020b7e7b1`

| 维度 | 值 |
|---|---|
| 名称 | `V2.3 大脑调度 - 30min tick` |
| 类型 | `agentTurn` (不是 shell) |
| 频率 | every 30 min |
| Agent | main (openclaw 大脑) |
| Session | isolated (主 session 不被污染) |
| Delivery | none (内部决策, 不外发) |
| 超时 | 300s |

### 11.3 大脑 6 步动作 (agentTurn message 模板)

1. **心跳 + 状态感知**: `GET /dev/agents/registry` → 看 4 角色 online
2. **拉 pending 任务**: `GET /dev/tasks?status=pending&limit=10`
3. **核心派任务**: `POST /dev/worker/team-dispatch {"limit": 5}` → matchTeammate 派给 claude-main/codex/deepseek
4. **leader 任务自己领**: `inferTaskType === 'plan'` → openclaw 自己接, 不派队友
5. **监控 in_progress**: 任务卡 > 30min → reset, > 2h → nudge
6. **输出决策报告**: 6 行格式, 24h 后压缩成心跳

### 11.4 验证记录 (2026-06-10 21:22)

**手动触发后实测**:
- 21:20 in_progress=5, pending=1037
- 21:22 in_progress=20, pending=1022
- **15 个 P1 任务从 pending → in_progress, 全 dispatched 给 claude-main**
- 0 任务新创建/删除, 0 任务虚假完成
- 任务类型: `[V4-ast] backend:xxx → 路由缺失认证` (V4 审计遗留 P1)

**说明**:
- ✅ 调度链工作正常: cron → openclaw 大脑 → team-dispatch API → 写 dev_spawn_queue
- ✅ matchTeammate 正确: P1 `bug` 类型 → `developer` 角色 → `claude-main`
- ✅ DB 路径正确: 没创建新任务, 没乱派 leader 任务
- ✅ 不靠 shell 脚本: 全程 openclaw 大脑决策

### 11.5 与 V5 对比

| 维度 | V5 (废弃) | V2.3 (当前) |
|---|---|---|
| 触发 | crontab shell | OpenClaw 内部 cron agentTurn |
| 调度 | Python 脚本 | openclaw 大脑调 API |
| 决策 | 脚本里 if/else | 大脑 LLM 推理 |
| 派任务 | spawn-queue.jsonl | team-dispatch HTTP API |
| 角色匹配 | 字面 title.includes | matchTeammate(inferTaskType) |
| 可见性 | 日志看脚本输出 | 决策报告 (V2.3 6 步) |
| 跨 OpenClaw 可移植 | ❌ 依赖 crontab 配置 | ✅ 任何 OpenClaw 配 cron 都能跑 |
| 飞书 delivery | 强依赖 webhook | 失效用 internal_messages 兜底 |

### 11.6 留给明天

1. 等 30 min 后看 cron 自动 tick 是否稳定 (lastRunStatus 必须 = ok)
2. 修 2 个 cron 的飞书 delivery target (70470ccb + 6cccfcce)
3. 决定 disabled 2 cron 删还是开 (207c6daa + 884fc21d)
4. 飞书 webhook 找新替换, 改用 internal_messages 兜底

### 11.7 永久铁律 (2026-06-10 海刚拍板, 写入 V2.3 MASTER)

```
调度 = OpenClaw 大脑主动调 API, 不是脚本定时
openclaw 大脑 = 调度中枢, 不领写代码任务
4 角色分工不可混: leader/developer/reviewer/auditor
P0 > P1 > P2 严格按优先级
P0 任务不许 skip, 即使 agent 不在也要等
不靠 MEMORY.md 记忆, 调度逻辑全在代码里
```
