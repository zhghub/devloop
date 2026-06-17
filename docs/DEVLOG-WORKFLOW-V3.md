# DevLog 开发中枢 — 系统工作模式 (V3.1 终态)

> **文档定位**: 本文是 DevLog 系统的"工作模式"权威描述,补充 `DEVLOG-V2.3-MASTER.md` 的"架构/数据/API"视角,聚焦"谁在什么时机做什么、产物流向何处"
>
> **版本**: V3.1 (2026-06-12 10:55 GMT+8) — 修订 2026-06-12 12:00: 全源码重读后状态同步 + P0 bug修复 + agent角色对齐
> **关键升级**: ① 方案来源扩展为 4 入口(后台+飞书+企微+微信) ② 智能审查升级为"代际进化"型(每轮自学习)
> **适用范围**: 任何 OpenClaw 实例 + ACP Agent 组合,都按本文档对齐工作流
> **配套**: 与 `DEVLOG-V2.3-MASTER.md` (架构) + `DEVLOG-MAP.md` (代码地图) 一起看

---

## 第一章: 终极目标 (最终形态)

DevLog 系统的**最终形态**是一个"自进化、自调度、自修复"的项目指挥中枢。任何一个新方案**通过任何通道**进入系统后,**无需人为干预**就能完成"分析→拆解→建表盘→派任务→审查→修复→复盘→再审查"的**自进化闭环**。

### 1.1 一句话目标

> **方案从 4 入口(后台/飞书/企微/微信) 进入 → DevLog 自主完成"方案解析→建项目表盘→智能审查(代际进化)→派修复任务→复盘迭代"全流程 → 每次审查都比上次更精准**

### 1.2 三个核心能力 (终态必达)

| 能力 | 一句话描述 | 终态标志 |
|---|---|---|
| **📥 4 入口方案接入** | 方案从管理后台/飞书/企微/微信 任意通道进入,统一接入 | 任意通道的方案都能触发同一个解析流程 |
| **🧠 代际进化智能审查** | 审查 agent **活的**,每轮看上次结果/任务完成率/准确度,自动调整下轮审查策略 | 第 N 轮审查质量分数 > 第 N-1 轮,自动可证 |
| **🔧 自修复闭环** | 审查结果自动转修复任务,修复 agent 完成后回到循环 | 审查 → 修复 → 验证 → 复盘 → **进化到下一轮审查** 闭环无停摆 |

---

## 第二章: 6 角色协同 (V3.1 终态)

### 2.1 角色定义

| # | 角色 | agent_name | CLI | acpx agentId | 职责 | 活度 |
|---|---|---|---|---|---|---|---|
| 1 | **leader** (调度中枢) | openclaw | (self) | openclaw | 接收 cron/手动触发,分派任务,不动代码 | ✅ 真活 |
| 2 | **intake-agent** (方案接入) | openclaw-intake | (openclaw LLM) | openclaw | 识别飞书/企微/微信/后台 来的方案,转 plan-parsing | ❌ 缺 |
| 3 | **plan-parsing-agent** (方案解析) | openclaw-plan | claude | claude | 解析用户上传的方案文档 → 模块/任务/优先级,建项目表盘 | ⚠️ 临时复用 openclaw |
| 4 | **review-agent** (代际审查) | codex | codex | codex | **代际进化**审查:读上次 generation,动态调整本次审查 | ⚠️ acpx 已配, 待 cron 真活验证 |
| 5 | **developer** (修复开发) | claude-main | claude | claude | 领取修复任务,编码/Bug 修复/重构/测试 | ✅ 真活 |
| 6 | **auditor** (深度审计) | mini-agent | mini-agent | mini-agent | 安全扫描/合规审计/深度分析 (替代已卸载的 deepseek-tui) | ⚠️ acpx 已配, 待真活验证 |
| 7 | **assistant** (辅助开发) | opencode | opencode | opencode | 多语言/REPL/片段辅助 | ⚠️ 不进主池 |
| 8 | **doc-agent** (文档/复盘) | openclaw-doc | (openclaw LLM) | openclaw | 任务完成后自动写 lessons/铁律/论坛复盘帖 | ❌ 缺 |

### 2.2 角色映射表 (任务类型 → 角色)

| 任务类型 (title 关键词 / priority / source) | 角色 | acpx agentId |
|---|---|---|---|
| 方案/规划/解析/analyze | plan-parsing-agent | claude |
| 接入/intake/来自通道 | intake-agent | openclaw |
| 全项目审查/模块审查/反向审查/审计/security | review-agent | codex |
| 安全扫描/合规审计/深度分析 | auditor | mini-agent |
| feature/bug/refactor/migrate | developer | claude |
| 架构调整/性能优化/BOM 大改/refactor | developer | claude |
| test/review/文档 | developer | claude |
| 文档/复盘/announcement/lesson | doc-agent | openclaw |
| 调度/通知/合并/triage | leader | openclaw |

---

## 第三章: 系统工作流 (V3.1 完整链路)

### 3.1 全景图

```
┌──────────────────────────────────────────────────────────────────────┐
│                方案 4 入口(本版本新增)                                  │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐        │
│  │ 管理后台    │ │ 飞书消息    │ │ 企微消息    │ │ 微信消息    │        │
│  │ ProjectCntr│ │ 私信/群     │ │ 智能机器人  │ │ lightclaw  │        │
│  └──────┬─────┘ └──────┬─────┘ └──────┬─────┘ └──────┬─────┘        │
└─────────┼─────────────┼─────────────┼─────────────┼───────────────┘
          ↓             ↓             ↓             ↓
   ┌────────────────────────────────────────────────────────┐
   │      intake-agent (新增) — 方案接入识别                  │
   │   - 通道识别 (4 入口)                                    │
   │   - 格式规范化 (Markdown 提取/附件下载/语音转文字)         │
   │   - 判定"是否方案" (长度 + 关键词 + LLM 兜底)            │
   │   - 转 plan-parsing-agent                                │
   └────────────────────────┬───────────────────────────────┘
                            ↓
   ┌────────────────────────────────────────────────────────┐
   │      OpenClaw 大脑中枢 (leader)                         │
   │   - 接收所有产物,决策下一步                              │
   │   - cron 定时触发 (V3.0 待修)                            │
   │   - 4 层硬约束保护 (✅ 已落地)                            │
   └────────────────────────┬───────────────────────────────┘
                            ↓
   ┌────────────────────────────────────────────────────────┐
   │      plan-parsing-agent (升级)                          │
   │   - 拆解方案: {modules, tasks, priority, module}        │
   │   - 自动建项目表盘 + 任务池                              │
   │   - 弹药库参考 (V3.1 新增)                              │
   └────────────────────────┬───────────────────────────────┘
                            ↓
   ┌────────────────────────────────────────────────────────┐
   │      DevLog 数据中枢                                    │
   │  ┌──────────────┐  ┌──────────────┐                      │
   │  │ 项目表盘      │  │ 任务表        │                     │
   │  │ project_*    │  │ dev_task_logs│                     │
   │  └──────────────┘  └──────────────┘                      │
   │  ┌──────────────┐  ┌──────────────┐                      │
   │  │ 弹药库        │  │ 铁律/经验     │                     │
   │  │ arsenal      │  │ guidelines   │                     │
   │  │              │  │ lessons      │                     │
   │  └──────────────┘  └──────────────┘                      │
   │  ┌──────────────┐  ┌──────────────┐                      │
   │  │ 论坛          │  │ 审查历史 (新) │                      │
   │  │ announcements│  │ audit_runs   │ ← 代际进化数据源     │
   │  └──────────────┘  └──────────────┘                      │
   └────────────────────────┬───────────────────────────────┘
                            ↓
   ┌────────────────────────────────────────────────────────┐
   │      review-agent (V3.1 重设计) 代际进化                │
   │   ┌────────────────────────────────────────────┐       │
   │   │  读 dev_audit_runs.generation=N-1 记录     │       │
   │   │  - quality_score (上次质量分)              │       │
   │   │  - findings_by_type (上次问题分布)         │       │
   │   │  - tasks_created (上次生成了多少修复任务)   │       │
   │   │  - details (上次审查发现的 5 维问题清单)    │       │
   │   └─────────────────┬──────────────────────────┘       │
   │                     ↓                                    │
   │   动态生成本次审查 prompt:                               │
   │   - 上次发现 X 类问题多 → 本次加深 X                    │
   │   - 上次生成 N 条任务, 完成率 Y% → 调整任务粒度         │
   │   - 上次评分 Z → 调整审查阈值                           │
   │                     ↓                                    │
   │   POST /project/review {scope, deep, generation: N}     │
   │   → 写回 dev_audit_runs.generation=N                     │
   └────────────────────────┬───────────────────────────────┘
                            ↓
   ┌────────────────────────────────────────────────────────┐
   │      自动转修复任务 (V3.0 P0-2)                          │
   │   findings → dev_task_logs (按 severity 自动 P0/P1/P2)  │
   └────────────────────────┬───────────────────────────────┘
                            ↓
   ┌────────────────────────────────────────────────────────┐
   │      developer 修复 (V3.0 流程 C)                        │
   └────────────────────────┬───────────────────────────────┘
                            ↓
   ┌────────────────────────────────────────────────────────┐
   │      doc-agent 复盘 (V3.0 P0-3/4)                       │
   │   - 写 lessons/铁律                                      │
   │   - 发论坛帖                                            │
   │   - 弹药库入库                                          │
   └────────────────────────┬───────────────────────────────┘
                            ↓
   ┌────────────────────────────────────────────────────────┐
   │      回到 review-agent 触发下一代审查                    │
   │   generation = N + 1                                    │
   │   看到 developer 改了什么 → 评估修复有效率               │
   │   → 调整下轮审查策略                                     │
   └────────────────────────────────────────────────────────┘
```

---

## 第四章: 6 条主工作流 (V3.1 终态)

### 流程 A: 方案接入 — 4 入口统一

```
入口 1: 管理后台
  ProjectCenter.vue → 上传 Markdown
  → /api/v1/dev/project/docs/upload
  → DB: project_docs
  → 调 plan-parsing-agent

入口 2: 飞书消息
  飞书长连接 → openclaw 收到消息
  → intake-agent 识别"这是方案"
    判定: startsWith('方案:') / startsWith('# ') / content.length > 500 / 包含"模块/任务/优先级"
  → intake-agent 格式规范化
  → 转 plan-parsing-agent (通过 /api/v1/dev/project/docs/analyze)

入口 3: 企微消息
  同入口 2 (路径: wecom → wecom-service → openclaw → intake)

入口 4: 微信消息
  同入口 2 (路径: lightclawbot → openclaw → intake)

    ↓
intake-agent 输出:
  {
    source_channel: 'feishu' | 'wecom' | 'wechat' | 'admin',
    source_user_openid: 'ou_xxx',
    content: '规范化后的 Markdown 文本',
    attachments: [{name, url}],
    received_at: timestamp
  }
    ↓
plan-parsing-agent (流程 A 下一阶段)
```

### 流程 B: 方案解析 → 项目表盘 → 任务

```
intake-agent 输出 → plan-parsing-agent
    ↓
plan-parsing-agent (V3.0 已有, V3.1 升级):
  1. 读 arsenal 弹药库 (V3.1 新增, 避免重复造轮子)
  2. LLM 拆解方案 → {modules, tasks, dependencies}
  3. 校验任务 4 段式 description
    ↓
后端自动:
  1. INSERT INTO project_modules (建项目表盘)
  2. INSERT INTO dev_task_logs (建原始开发任务, 4 段)
  3. UPDATE project_versions (新版本 vN)
    ↓
通知 intake-agent 反馈给用户:
  "✅ 方案已解析, 创建了 5 个模块 + 12 个任务
   项目表盘: /erp/dev/ProjectCenter?projectId=xxx"
```

### 流程 C: 代际进化智能审查 (V3.1 重点)

```
触发方式 1: 定时 (P0-1)
  openclaw cron 677d03ce 改写后
  → 每天 03:00 跑一次项目全盘审查 (scope=project, deep=true)
  → 每周一 09:00 跑模块审查 (按 dev_module_registry.completion_pct < 80% 选)

触发方式 2: 手动
  ProjectCenter.vue → "项目全盘审查" / "模块审查"
  → POST /api/v1/dev/project/review {scope, deep, module?}

触发方式 3: 流程 C 修复完成 (V3.1 新增闭环)
  developer 完成修复任务
  → doc-agent 复盘完成
  → 自动触发 review-agent 验证修复 (scope=module, module=xxx, deep=false)
  → quality_score 评分, 反馈到 generation 进度
```

**review-agent 工作机制 (代际进化核心)**:

```python
# 伪代码, V3.1 待实现
async def generation_review(scope, deep, target=None):
    # 1. 读上次 generation 数据
    last_run = await db.query("""
        SELECT * FROM dev_audit_runs
        WHERE scope = $1 AND target = $2
        ORDER BY generation DESC LIMIT 1
    """, [scope, target])
    
    generation = (last_run.generation or 0) + 1
    
    # 2. 根据上次数据动态调整本次审查策略
    if last_run:
        prompt = f"""
        # 你是审查专家, 这是你的第 {generation} 代审查
        
        ## 上次审查结果 (代际 N-1)
        - 质量分: {last_run.quality_score}
        - 问题分布: {last_run.findings_by_type}  # {security: 5, performance: 3, ...}
        - 生成修复任务: {last_run.tasks_created} 条
        - 任务完成率 (查 dev_task_logs 反馈): {completion_rate}%
        
        ## 本次审查策略调整
        - 如果上次 security 类问题多, 本次深挖安全
        - 如果上次生成的任务完成率 < 50%, 本次任务粒度细化
        - 如果上次评分 < 60, 扩大审查范围
        
        ## 现在的审查目标
        {scope} {target} deep={deep}
        
        ## 五维审查
        1. 安全性 2. 正确性 3. 性能 4. 可维护性 5. 合规性
        
        请输出 JSON {{findings, score, summary, strategy_used}}
        """
    else:
        generation = 1
        prompt = f"首次审查 {scope} {target}, 五维标准..."
    
    # 3. 派 codex/deepseek 跑审查
    result = await spawn_agent('codex', 'review-agent', prompt)
    
    # 4. 写回 dev_audit_runs
    await db.execute("""
        INSERT INTO dev_audit_runs (generation, scope, target, started_at, duration_ms,
            findings_count, findings_by_type, tasks_created, quality_score, details)
        VALUES ($1, $2, $3, NOW(), $4, $5, $6, $7, $8, $9)
    """, [generation, scope, target, ..., result.score, ...])
    
    return result
```

### 流程 D: 修复任务派发

```
任务来源 (V3.1 扩展):
  - 流程 A 生成的原始任务
  - 流程 B 生成的审查修复任务 (findings → dev_task_logs)
  - 流程 C 修复完成后的验证任务 (闭环)
  - 手动发

    ↓
OpenClaw leader cron (677d03ce):
  1. 4 层硬约束
  2. 6 角色匹配 (V3.1 扩展)
  3. POST /api/v1/dev/worker/team-dispatch
    ↓
spawn_queue → PG NOTIFY → pg-listen-daemon (实时, 12-55ms)
    ↓ HTTP wake-dispatch
OpenClaw sessions_spawn(runtime="acp") → 真活 agent
    ↓ ACP 协议
developer / reviewer / auditor 完成
    ↓
POST /worker/spawn-result (auto lessons + 复盘帖)
```

### 流程 E: 复盘 → lessons + 论坛 + 弹药库 (V3.1 强化)

```
修复任务 completed
    ↓
doc-agent 接收 (V3.0 缺, V3.1 必须建):
  1. 读 task_id 的 learnings/remaining/issues
  2. 提炼 1-2 条 lessons → POST /lessons (47 类, 14 严重度, 4 通道)
  3. 重要经验升级铁律 → POST /guidelines
  4. 发论坛复盘帖 → POST /announcements (V3.1-forum 风格)
  5. 任务用的工具入库 → POST /arsenal (V3.1 必须修 0 行后端)
  6. 写一份本次复盘的"审查策略建议" → dev_audit_runs.feedback 字段 (V3.1 新增)
    ↓
管理后台 Announcements/ArsenalManager/Guidelines.vue 自动展示
```

### 流程 F: 项目表盘自演化

```
每完成 1 个修复任务:
  1. dev_task_logs.completed_at 更新
  2. project_modules.completion_pct 重新计算 (V3.1 自动)
  3. project_changelog 追加 (V3.1 自动)
  4. project_versions 自动探测最新 commit (V3.1 自动)
    ↓
[自动触发流程 C 验证审查, 启动下一代审查循环]
```

---

## 第五章: 与 V2.3 现状对账 (差距清单 V3.1)

> V3.0 → V3.1 新增了 2 大类: 4 入口接入 + 代际进化审查。下面分两段对账。

### 5.1 V3.0 差距回顾 (24 项 → 12 项完成) + V3.1 新增

> **2026-06-12 全源码验证**: V3.0 24 项中约 12 项已完成落地 (三级降级链/同模块聚类/enqueue原子化/acpx_session_id列/agent-check异步化/agent全员/probeAgent/4段渐进校验/expert-panel/pipeline/spawn-result端点)

| 流程 | V3.0 差距 | 已完成 | 剩余 | V3.1 新增 | 总剩余 |
|---|---|---|---|---|---|
| A 方案→表盘→任务 | 2 | 0 | 2 | 0 | 2 |
| B 智能审查 | 6 | 3 | 3 | **8** | 11 |
| C 修复派发 | 2 | 2 | 0 | 0 | 0 |
| D 复盘 | 5 | 0 | 5 | 0 | 5 |
| E 弹药库 | 6 | 0 | 6 | 0 | 6 |
| F 表盘自演化 | 3 | 0 | 3 | 0 | 3 |
| **V3.1 新增: 4 入口接入** | — | — | — | **4** | 4 |
| **总计** | **24** | **~12** | **~12** | **12** | **~24** |

### 5.2 V3.1 12 项新差距 (清单)

#### 4 入口接入 (4 项)

| # | 差距 | 影响 | 修法预估 |
|---|---|---|---|
| INTAKE-1 | intake-agent 角色没建 (不存在) | 飞书/企微/微信发的方案会被 LLM 兜底乱答 | 4h (新建 agent 角色 + message-entry 加 startsWith 规则) |
| INTAKE-2 | message-entry.js 没"方案"识别 (8 个硬编码全是审核/部门) | 飞书发"方案:xxxxx"被当成普通聊天 | 1h (加 startsWith + 长度判定) |
| INTAKE-3 | 通道 → project/docs/upload 直通链路缺 | 即便识别了方案,怎么把飞书消息转给 plan-parsing? | 2h (写 message → dev/project/docs/upload 桥) |
| INTAKE-4 | 附件下载/语音转文字没接 | 飞书发的图、PDF、语音没法解析 | 4h (调 MiniMax VL-01 视觉 + ASR) |

#### 代际进化审查 (8 项)

| # | 差距 | 影响 | 修法预估 |
|---|---|---|---|
| GEN-1 | review.js 读不到上次 generation 数据 | 审查永远是"首查" | 2h (review.js 加读 dev_audit_runs 上次记录) |
| GEN-2 | 没"代际"概念字段, generation 是 number 但没人维护 | 代际永远 1 | 1h (写 generation 自增逻辑) |
| GEN-3 | 审查 prompt 不带"上次结果" | 审查 agent 看不到上下文 | 2h (review.js prompt 模板加 generation 上下文) |
| GEN-4 | 5 维审查 (安全/正确/性能/可维护/合规) 没定义 schema | agent 每次返不同 JSON | 1h (加 JSON schema 校验) |
| GEN-5 | 审查 → 任务 自动转换没做 (V3.0 P0-2) | 审查结果要手动建任务 | 3h (review.js 加 autoCreateTasks) |
| GEN-6 | 任务完成率/准确度没回写到 dev_audit_runs | 下次审查看不到上轮任务反馈 | 2h (spawn-result 处理器加 audit 反馈) |
| GEN-7 | dev_audit_runs.details 没被审查 agent 解析 | 5 维问题清单只是 JSON 存, 没用 | 1h (V3.0 已有表, V3.1 加解析) |
| GEN-8 | 代际进化的 quality_score 计算公式没定义 | 怎么算"质量分"靠拍脑袋 | 1h (写 score = weighted(finding_count, severity, task_completion_rate)) |

### 5.3 差距汇总 V3.1 修订 (P0/P1/P2)

| 优先级 | V3.0 剩余 | V3.1 新增 | 总数 | 备注 |
|---|---|---|---|---|
| P0 | 2 (cron改写/审查→任务) | 4 (INTAKE-1/2/3, GEN-1) | **6** | V3.0 的 4 个P0已完成2个 |
| P1 | 6 | 6 | 12 | V3.0 13个中7个已完成 |
| P2 | 4 | 2 | 6 | V3.0 7个中3个已完成 |
| **总** | **12** | **12** | **24** | V3.0 原24项→剩12项 |

### 5.4 V3.1 4 个新 P0 (优先做)

| # | 差距 | 修法 |
|---|---|---|
| P0-5 | intake-agent 缺失 | message-entry 加 4 通道"方案"识别 startsWith + length 判定 |
| P0-6 | 飞书/企微/微信"方案"消息乱答 | 改 message-entry 的兜底逻辑, 不再 LLM 自由发挥 |
| P0-7 | 4 通道 → plan-parsing 桥缺失 | 写 message → /api/v1/dev/project/docs/upload 转接 |
| P0-8 | review agent 不带上次结果 | review.js prompt 模板读 dev_audit_runs 上次记录 + generation 自增 |

### 5.5 V3.1 代际进化数据流 (3 闭环)

```
┌──────────────────────────────────────────────────┐
│  闭环 1: 审查→修复→再审查 (任务粒度进化)         │
│  generation=N: 审查发现 5 个 security             │
│  → 生成 5 个 P0 修复任务                          │
│  → developer 修复 3 个 (完成率 60%)              │
│  → generation=N+1 审查:                          │
│    prompt: "上次 security 有 5 个, 修了 3 个,     │
│            剩 2 个请重点验证 + 看是否引入新漏洞"   │
│  → quality_score: 70 → 80 (进化了)              │
└──────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────┐
│  闭环 2: 审查→任务粒度进化                        │
│  generation=N: 生成 10 任务, 修了 4 (40%)         │
│  → 任务粒度太粗, 完成率低                        │
│  → generation=N+1 审查:                         │
│    prompt: "上次任务完成率仅 40%, 本次拆分更细,  │
│            每个发现独立任务, 单任务 < 4h"         │
│  → quality_score: 70 → 78 (进化了)              │
└──────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────┐
│  闭环 3: 审查→准确度进化                          │
│  generation=N: 审查出 10 个问题                   │
│  → 后续发现其中 3 个是误报                        │
│  → task_completion_rate (修了的/真问题) = 70%    │
│  → generation=N+1 审查:                         │
│    prompt: "上次准确度 70%, 有 3 个误报,          │
│            本次在以下方面更严格: <列具体标准>"     │
│  → quality_score: 70 → 85 (进化了)              │
└──────────────────────────────────────────────────┘
```

### 5.6 dev_audit_runs 表已有, V3.1 怎么用

```sql
-- 表结构 (V2.3 已有, V3.1 补数据)
-- generation    INT        -- 代际, 自增 (V3.1 必须维护)
-- scope         -- 'project' | 'module' | 'code'
-- target        -- 模块名/路径
-- started_at    TIMESTAMP
-- duration_ms   INT
-- findings_count INT
-- findings_by_type JSONB  -- {security: 5, performance: 3, ...}
-- tasks_created  INT      -- 本次生成了多少修复任务
-- quality_score  FLOAT    -- 0-100, 衡量审查质量 (V3.1 加公式)
-- details        JSONB    -- 5 维问题清单 + strategy_used

-- V3.1 查询: 找某个模块的最近一次审查
SELECT * FROM dev_audit_runs
WHERE scope = 'module' AND target = 'product'
ORDER BY generation DESC LIMIT 1;

-- V3.1 计算: 任务完成率
SELECT COUNT(*) FILTER (WHERE status = 'completed') * 100.0 / COUNT(*) AS completion_rate
FROM dev_task_logs
WHERE source = 'audit' AND created_at BETWEEN $audit_run_started_at AND NOW();
```

---

## 第六章: V3.1 落地路线图

### 6.1 V3.1 立即做 (本周, 6 P0 + 配置就绪项 共 15h)

| 序 | 任务 | 工作量 | 优先级 | 状态 |
|---|---|---|---|---|
| 1 | ✅ PG LISTEN 守护 (替代 acpx-spawn-consumer.sh): LISTEN→claim→wake-dispatch 闭环 | 3h | P0 | ✅ 已完成 (2026-06-12) |
| 2 | ✅ sessions_spawn 端到端调通: NOTIFY→LISTEN→dispatch→spawn-result 全链路 | 2h | P0 | ✅ 已完成 (35 dispatched, 0 failed) |
| 3 | intake-agent 4 入口识别 (P0-5/6/7) | 7h | P0 | ❌ 全缺 |
| 4 | review.js 代际 prompt 改造 (P0-8) — GEN-1/3 已完成 (review.js L184-208), GEN-2/4/5/6/7/8 待做 | 4h | P0 | 🟡 部分完成 |
| 5 | ✅ 自动 lessons 写 (spawn-result §6 触发) | 2h | P0 | ✅ 已完成 (2026-06-14) |
| 6 | ⛔ 自动复盘帖 — **海刚 6/14 17:55 铁律暂不做** (见下方复盘帖铁律) | 1h | ~~P0~~ 暂搁 | ⛔ DISABLED (spawn-queue.js:444 已 `&& false`) |
| ✅ | 弹药库后端 0 行修 (V3.0 P0-3) | — | P0 | ✅ 待验证 |
| ✅ | openclaw cron 后端就绪 (V3.0 P0-1) | — | P0 | ✅ 就绪 (见上#1) |
| ✅ | 审查→任务自动转换 (V3.0 P0-2) | — | P0 | ✅ review.js 已有模式 |

> **复盘帖铁律 (2026-06-14 17:55 海刚强化)**: 任务完成**只去** `dev_task_logs` + `dev_lessons_learned`,**不再发论坛**。原意图 (V3.1 P0-4) 已被禁。回滚方式: spawn-queue.js:444 把 `false /* DISABLED */` 改成 `true /* ENABLED */`。**暂不做复盘帖设计**,等想清"复盘怎么不发成论坛噪音"再恢复 (feedback-no-retro-post)。

### 6.2 V3.1 短期 (2 周, 19 P1 共 35h)

按 8 个 P1 领域并行做:
- INTAKE 4 项补完 (附件下载/语音转文字)
- GEN 5 项补完 (5 维 schema/任务粒度/准确度回写/score 公式/feedback 字段)
- 表盘自演化 3 项 (completion_pct trigger / changelog / versions auto)
- 弹药库 5 项 (CRUD/分类/搜索/推荐)

### 6.3 V4.0 中期 (1 个月, 9 P2)

- 专用 6 角色独立 agent (不复用 openclaw)
- 代际进化的 quality_score 训练数据采集 (准备 50+ 轮)
- 弹药库智能推荐 (plan-parsing 时自动选技术栈)
- 项目表盘 3D 仪表盘 / 时序图

---

## 附录 A: 关键路径/文件 (V3.1 更新)

| 关注点 | 位置 | V3.1 状态 |
|---|---|---|
| **4 入口接入** (V3.1 新) | `message-entry.js` (待加识别) + 新建 `intake-agent.js` | ❌ 待建 |
| 项目表盘前端 | `ProjectCenter.vue` | ✅ 完整 |
| 项目表盘后端 | `routes/_split/project-center/common/*` (14 端点) | ✅ |
| 审查引擎 | `routes/review.js` + `services/ai-audit-engine.js` | ⚠️ V3.1 需加 generation 上下文 |
| **代际进化数据源** (V3.1 新) | `dev_audit_runs` 表 (4 条已有) + `dev_audit_history` 表 (12 条已有) | ✅ 表在, 用法待升级 |
| 弹药库前端 | `ArsenalManager.vue` | ⚠️ 调空端点 |
| 弹药库后端 | `routes/arsenal-data.js` | ❌ **0 行** |
| 论坛前端 | `Announcements.vue` | ✅ 已是 V3.1-forum |
| 论坛后端 | `dev_announcements` 表 | ✅ |
| 铁律前端 | `Guidelines.vue` | ✅ |
| 铁律后端 | `dev_guidelines` 表 (48 条) + `routes/guidelines.js` | ⚠️ 端点返 0 |
| 经验教训 | `dev_lessons_learned` 表 (1479 条) | ✅ 数据多 |
| **消息通道入口** (V3.1 关键) | `openclaw-permission-gateway/src/register-flow.ts` (L 注册) + `message-entry.js` (业务路由) | ⚠️ 8 硬编码 startsWith, 无方案识别 |


## 附录 A-2: V3.0 新增/变更文件 (2026-06-11 ~ 2026-06-12)

### 新增文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `routes/spawn-queue.js` | 169 | +spawn-result 端点 (含 PostgreSQL type CAST 修复 T20260611-14901) |
| `routes/expert-panel.js` | 201 | Expert Panel (acpx compare 模式) + Pipeline (acpx flow run 模式) |
| `migrations/20260611_devlog_v3_acpx_session.sql` | 9 | acpx_session_id + session_status 列 |
| `services/_split/team-dispatcher/pipeline.js` | 77 | 派发前 5 段流水线 (FP→4段→排序→聚类) |
| `services/_split/team-dispatcher/roles.js` | 173 | 角色映射/matchTeammate/inferTaskType/buildDelegationPrompt |
| `services/_split/team-dispatcher/spawn.js` | 182 | loadAgentApiEnv/spawnAcpxAgent/enqueueSpawn |
| `services/_split/team-dispatcher/validation.js` | 131 | FP预检/4段校验/rejectIncompleteFourSectionTasks |

### 重构文件

| 文件 | 旧行数 | 新行数 | 说明 |
|------|--------|--------|------|
| `services/team-dispatcher.js` | 669 | 298 | facade: teamDispatch + orphanStaleInProgress (其余拆入 _split/) |
| `routes/agent-check.js` | 19 | 45 | execSync→execAsync + truthy.probeAgent 统一探测 |
| `frontend/DevLogs.vue` | 284 | 370 | +Agent 实时状态面板 (5 Agent 卡片 + 心跳 + 当前任务) |

### 配置变更

| 文件 | 变更 |
|------|------|
| `~/.acpx/config.json` | Claude +IS_SANDBOX=1; 新增 mini-agent 条目 |
| `.acpxrc.json` | 项目级 acpx 默认配置 (defaultAgent=claude, approve-all) |
| `docker-compose.yml` | +/host-nvm:ro mount; +/usr/local/bin mount |
| OpenClaw config | plugins.entries.acpx.config.permissionMode=approve-all |

### 已删除/迁移

| 文件 | 操作 |
|------|------|
| `scripts/spawn-consumer-v23.py` | ✅ git rm (6/15 17:05 海刚 STOPPED-0615-1557) |
| `scripts/scheduler-v23.py` | ✅ git rm (6/11 25fe442d V3.0 铲除) |
| `scripts/devlog-auto-dispatch.sh` | ✅ git rm (6/15 17:22 V3.1 修订文档 #10) |
| `scripts/spawn-consumer-s1-check.sh` | ✅ git rm (6/15 17:05 V23 配套) |
| `scripts/devlog-team-dispatch-cron.sh` | ✅ git rm (6/15 17:05 机械拉 cron) |
| `scripts/minimax-heartbeat.sh` | ✅ git rm (6/15 17:05 机械 1min) |
| `scripts/devlog-dispatch.py` | ✅ git rm (6/11 25fe442d V3.0 铲除) |
| `scripts/devlog-audit-deepseek.py` | ✅ git rm (6/11 25fe442d) |
| `backend/src/modules/devlog/scripts/devlog.sh` | ✅ git rm (6/15 17:22 重复) |
| `backend/src/modules/devlog/services/_split/s1-checker.js` | ✅ git rm (6/15 17:30 V23 S1 自检, 后端 0 引用) |
| `services/_split/ai-audit/{audit,prompt,verdicts,fallback}.js` | 删除 (孤儿子文件) |

> **2026-06-15 清理状态**: V2.0 / V2.3 / V3.0 时代机械调度脚本全部 git rm (10 项).
> V3.1 范式: OpenClaw 大脑 → POST team-dispatch → enqueueSpawn+NOTIFY → 宿主机 daemon (`devlog-pg-listen.service`) LISTEN → claim → spawn acpx → reap.

### 已知 Bug (已修复)

| Bug | 文件 | 状态 |
|-----|------|:---:|
| require('../../../../../lib/db') 多一级 .. → 容器启动失败 | validation.js:14, spawn.js:14 | ✅ 已修复为 ../../../../lib/db |

### 已知 Bug (待修)

| Bug | 位置 | 优先级 |
|-----|------|:---:|
| TEAM_ROLES.auditor=deepseek 但 TRUTHY_ROLES.auditor=mini-agent (不一致) | roles.js vs truthy.js | P1 |
| TRUTHY_ROLES mini-agent key 重复 | truthy.js:80,88 | P2 |
| workboard-routes-v5.js 标记 Stage 2 待删 | 58 行残留 | P2 |
| analyze-v2-handler.js 显式 @deprecated 零引用 | 76 行死代码 | P2 |

## 附录 A-3: 当前实际 Agent 配置 (acpx + 代码 双重验证)

| agent_name | TEAM_ROLES key | TRUTHY_ROLES key | acpx agentId | acpx config | IS_SANDBOX | API 协议 | 状态 |
|---|---|---|---|---|---|---|---|
| openclaw | leader | leader | openclaw | ✅ self | N/A | — | ✅ |
| claude-main | developer | developer | claude | ✅ | ✅ | MiniMax /anthropic | ⚠️ 待真活 |
| codex | reviewer | reviewer | codex | ✅ | — | MiniMax /v1 | ⚠️ 待真活 |
| mini-agent | mini-agent | auditor + mini-agent | mini-agent | ✅ | — | MiniMax /anthropic | ⚠️ 待真活 |
| opencode | assistant | assistant | opencode | ✅ | — | MiniMax 双协议 | ⚠️ 待真活 |
| deepseek | auditor (TEAM_ROLES) | — | deepseek | ❌ 未配 | — | — | ❌ 已卸载 |

## 附录 B: V3.1 P0 6 项总清单 (修订)

| # | 任务 | 涉及文件 | 工作量 |
|---|---|---|---|
| P0-1 | 定时智能审查 | openclaw cron 改写 + review.js cronTick | 2h |
| P0-2 | 审查→任务自动转换 | review.js autoCreateFixTasks | 3h |
| P0-3 | 自动 lessons 写 | spawn-result 处理器 | 2h |
| P0-4 | 自动复盘帖 | spawn-result 处理器 | 1h |
| **P0-5** | **intake-agent 4 入口识别** | **message-entry.js startsWith** | **4h** |
| **P0-6** | **飞书/企微/微信方案消息不乱答** | **message-entry.js 兜底改写** | **2h** |
| **P0-7** | **4 通道→plan-parsing 桥** | **新写 message→project/docs/upload** | **2h** |
| **P0-8** | **review agent 带上次结果** | **review.js prompt 加 generation** | **4h** |
| **小计** | | | **20h** |

## 附录 C: 跟"最终目标"的距离 (V3.1)

> 当前 DevLog 离"4 入口接入 + 代际进化智能审查 + 自修复"项目指挥中枢, 还差 **36 项 + 20h P0 任务**。

> **核心结论 (V3.1 2026-06-12 修订)**:
> - **流程 A (方案接入)**: 60% 完成, **缺 4 入口 + intake-agent**
> - **流程 B (方案解析)**: 80% 完成
> - **流程 C (代际审查)**: 35% 完成 (+5%, dev_audit_runs 表 + review.js 模式就绪; 代际进化 prompt 待做)
> - **流程 D (派发)**: **95% 完成** (+5%, 三级降级链/同模块聚类/enqueue原子化/spawn-result端点已落地)
> - **流程 E (复盘)**: 40% 完成
> - **流程 F (弹药库)**: 30% 完成
> - **流程 G (表盘自演化)**: 60% 完成

> **关键瓶颈 V3.1 修订**:
> 1. **方案入口只 1/4 (管理后台)**, 飞书/企微/微信 0%
> 2. **审查 agent 是"死的"** — 每次审查独立, 不读上次结果
> 3. **完成后没自动写 lessons/铁律/论坛** (V3.0 老问题)

---

_本文档 V3.1 写于 2026-06-12 10:55 GMT+8, 作者 openclaw-main, 海刚反馈"4 入口 + 代际进化"修订。_
_2026-06-12 12:00 GMT+8 修订: Claude Code 全源码重读后状态同步 — agent角色对齐/差距清单更新/新增V3.0变更记录/已知bug清单。_
