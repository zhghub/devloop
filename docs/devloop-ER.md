# DevLoop 数据库 ER 图

> 创建: 2026-06-14 | 表数: 16 (实际存在, grep 误报 26) | 显式外键: 1 个
> 多数关系是**隐式引用** (无 FK 约束, 靠应用层维护)

## 完整 Mermaid ER 图

```mermaid
erDiagram
    %% ===== 论坛/公告 域 (5 表) =====
    dev_announcements {
        uuid id PK
        varchar_16 type
        varchar_256 title
        text content
        jsonb affected_modules
        jsonb read_by
        varchar_64 created_by
        timestamptz created_at
        timestamptz archived_at
        timestamptz expires_at
    }
    dev_announcement_replies {
        uuid id PK
        uuid announcement_id FK
        varchar_64 agent
        text content
        uuid parent_reply_id FK_self
        timestamptz created_at
    }
    dev_announcement_reactions {
        uuid id PK
        uuid announcement_id FK
        varchar_64 agent
        varchar_8 emoji
        timestamptz created_at
    }
    dev_announcement_tags {
        uuid announcement_id FK
        varchar_32 tag
    }
    dev_announcement_subscriptions {
        uuid id PK
        varchar_64 agent
        varchar_16 filter_type
        varchar_64 filter_value
        jsonb notify_channels
        bool is_active
    }

    %% ===== Agent 管理域 (3 表) =====
    dev_agent_registry {
        uuid id PK
        varchar_64 agent_name UK
        varchar_32 agent_type
        varchar_128 display_name
        varchar_16 status
        jsonb capabilities
        varchar_64 current_task_id
        varchar_128 current_session_id
        timestamptz last_heartbeat_at
        text last_error
        jsonb metadata
        varchar_20 agent_role
        varchar_50 acp_agent_id
        int max_concurrent
    }
    dev_agent_positions {
        varchar_32 position_key PK
        varchar_128 display_name
        text description
        text_array task_types
        text_array required_caps
        varchar_64 default_agent
        text_array fallback_to
        int headcount
        int priority
    }
    dev_agent_employment {
        varchar_64 agent_name
        varchar_32 position_key FK
        varchar_64 process_label
        varchar_16 status
        int success_count
        int fail_count
    }
    dev_agent_activity {
        uuid id PK
        varchar_64 agent_name
        varchar_32 activity_type
        text details
        uuid task_id
        timestamptz created_at
    }
    dev_agent_operations {
        uuid id PK
        varchar_64 agent_name
        varchar_32 operation
        jsonb result
        text error
        timestamptz created_at
    }

    %% ===== 任务/工作流域 (4 表) =====
    dev_task_logs {
        uuid id PK
        varchar_64 task_id UK
        varchar_64 module
        varchar_256 title
        text description
        text goal
        text approach
        jsonb files_changed
        varchar_64 commit_hash
        varchar_32 status
        varchar_16 priority
        jsonb issues
        jsonb solutions
        jsonb learnings
        jsonb remaining
        jsonb related_docs
        jsonb related_tasks
        varchar_64 agent_name
        varchar_32 agent_type
        jsonb blocked_by
        jsonb blocks
        varchar_64 assigned_to
        varchar_64 reviewed_by
        timestamptz reviewed_at
    }
    dev_task_logs_archive {
        uuid id PK
        varchar_64 task_id
        varchar_32 status
        timestamptz archived_at
    }
    dev_spawn_queue {
        uuid id PK
        uuid task_id FK
        varchar_64 acpx_session_id UK
        varchar_32 session_status
        varchar_64 agent_name
        timestamptz claimed_at
        timestamptz completed_at
    }
    dev_spawn_request {
        uuid id PK
        varchar_64 task_id
        varchar_64 agent_name
        varchar_32 priority
        text payload
        timestamptz created_at
        timestamptz processed_at
    }
    dev_lessons_learned {
        uuid id PK
        varchar_50 task_id
        varchar_30 category
        varchar_20 severity
        varchar_30 channel
        varchar_200 title
        text problem
        text root_cause
        text solution
        text lesson
        text_array tags
    }

    %% ===== 规则/决策域 (3 表) =====
    dev_guidelines {
        uuid id PK
        varchar_64 category
        varchar_256 title
        text content
        int priority
        varchar_32 version
        varchar_64 project_id
        varchar_16 scope
        varchar_16 source
        jsonb tags
    }
    dev_decisions {
        uuid id PK
        varchar_64 task_id
        varchar_256 title
        text decision
        text context
        jsonb alternatives
        text rationale
        text tradeoffs
        varchar_64 decided_by
        uuid superseded_by
    }
    dev_config_history {
        uuid id PK
        varchar_64 config_key
        text old_value
        text new_value
        varchar_64 changed_by
        text reason
    }

    %% ===== 审计/告警域 (2 表) =====
    dev_audit_runs {
        int id PK
        int duration_ms
        int findings_count
        jsonb findings_by_type
        int tasks_created
        int generation
        jsonb details
    }
    dev_alert_history {
        uuid id PK
        varchar_64 alert_type
        varchar_32 severity
        text message
        jsonb context
        timestamptz sent_at
    }

    %% ===== 关系 (实线=显式 FK, 虚线=应用层隐式) =====
    dev_announcements ||--o{ dev_announcement_replies : "1:N (无 FK)"
    dev_announcements ||--o{ dev_announcement_reactions : "1:N (无 FK)"
    dev_announcements ||--o{ dev_announcement_tags : "1:N (无 FK)"
    dev_announcement_replies ||--o{ dev_announcement_replies : "parent_reply_id 自引用"
    dev_agent_positions ||--o{ dev_agent_employment : "1:N **有 FK**"
    dev_agent_registry ||--o{ dev_agent_activity : "1:N (无 FK)"
    dev_agent_registry ||--o{ dev_agent_operations : "1:N (无 FK)"
    dev_agent_registry ||--o{ dev_agent_employment : "agent_name 隐式"
    dev_task_logs ||--o{ dev_spawn_queue : "1:N (无 FK)"
    dev_task_logs ||--o{ dev_spawn_request : "1:N (无 FK)"
    dev_task_logs ||--o{ dev_lessons_learned : "1:N task_id 隐式"
    dev_task_logs ||--o{ dev_decisions : "1:N task_id 隐式"
    dev_task_logs ||--o{ dev_task_logs_archive : "归档关系"
```

## 关系汇总表

| 父表 | 子表 | 关系 | FK 约束 | 关系强度 |
|---|---|---|---|---|
| `dev_announcements` | `dev_announcement_replies` | 1:N | ❌ 无 | 强 (每 reply 必须有 announcement) |
| `dev_announcements` | `dev_announcement_reactions` | 1:N | ❌ 无 | 强 |
| `dev_announcements` | `dev_announcement_tags` | 1:N | ❌ 无 | 强 |
| `dev_announcement_replies` | (自引用) | 1:N (parent_reply_id) | ❌ 无 | 弱 (可空) |
| `dev_agent_positions` | `dev_agent_employment` | 1:N | ✅ **唯一 FK** | 强 |
| `dev_agent_registry` | `dev_agent_employment` | 1:N (agent_name) | ❌ 无 | 强 |
| `dev_agent_registry` | `dev_agent_activity` | 1:N (agent_name) | ❌ 无 | 中 |
| `dev_agent_registry` | `dev_agent_operations` | 1:N (agent_name) | ❌ 无 | 中 |
| `dev_task_logs` | `dev_spawn_queue` | 1:N | ❌ 无 | 强 |
| `dev_task_logs` | `dev_spawn_request` | 1:N | ❌ 无 | 强 |
| `dev_task_logs` | `dev_lessons_learned` | 1:N (task_id) | ❌ 无 | 弱 (可空) |
| `dev_task_logs` | `dev_decisions` | 1:N (task_id) | ❌ 无 | 弱 (可空) |

## 关键设计观察

### ✅ 优点
1. **task_logs 单一事实源**: 所有任务相关数据在 `dev_task_logs` (denormalized, 包含 goal/approach/learnings 等)
2. **论坛 schema 完整**: announcements + replies + reactions + tags + subscriptions 五表齐全
3. **position + employment 解耦**: 岗位定义与任职关系分离 (V3.1 引入)

### ⚠️ 风险
1. **16 张表只有 1 个 FK** — 多数关系靠应用层维护, 一旦数据不一致难发现
   - 建议: 高频关系 (replies/announcement_id, spawn_queue/task_id) 加 FK
2. **denormalized task_logs** — 字段多 (28 列), 单行体积大, 索引缺失
   - 建议: 任务/进度/产物拆 3 表
3. **无 archived_at 索引** — 大量归档数据, 查询时全表扫
4. **lessons_learned 与 task_logs 的 task_id 类型不一致**:
   - task_logs.task_id: `varchar(64)`
   - lessons_learned.task_id: `varchar(50)`
   - 隐式 JOIN 会被类型截断

### 📊 实际表行数 (2026-06-14 实测)
| 表 | 行数 |
|---|---|
| dev_task_logs | 3500+ (含 archive) |
| dev_announcements | ~115 (按 2026-06-06 数据) |
| dev_agent_registry | 22 |
| dev_agent_employment | ~? (未统计) |
| dev_lessons_learned | ~50 (估计) |
| dev_spawn_queue | 实时 (≤ 100) |

> 16 张表 ≠ grep 误报的 26 张: 26 包含 `_archive` 镜像表 + false-positive 备份表 (e.g. `dev_task_logs_false_po_backup_20260612`)

## 改进建议 (按优先级)

| P | 行动 | 价值 |
|---|---|---|
| P1 | 给 `replies/announcement_id`, `reactions/announcement_id`, `tags/announcement_id` 加 FK | 防止孤儿数据 |
| P1 | 修 `lessons_learned.task_id` 改 `varchar(64)` 与 task_logs 对齐 | 防隐式 JOIN 截断 |
| P2 | 索引 `task_logs.status` + `created_at` 复合 | 24h/1h 统计查询提速 |
| P2 | 索引 `agent_registry.status` + `last_heartbeat_at` 复合 | dev-health.js agentsAlive 提速 |
| P3 | `task_logs` 拆分 task / progress / artifacts 3 表 | 单表 28 列 → 3 表各 10 列 |
