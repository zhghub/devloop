# DevLog API Changelog (V2.3 → V3.1)

> 创建: 2026-06-14 | 数据源: DEVLOG-V2.3-MASTER.md / DEVLOG-MAP.md / DEVLOG-WORKFLOW-V3.md / 源码实读
> 维护方式: 每次发版 (V3.2/V4.0) 时增量追加, 旧版本冻结不删

## 📊 总体规模

| 版本 | 端点数 | 新增 | 移除 | 关键变化 |
|---|---:|---:|---:|---|
| V2.3 (2026-06-09) | ~90 | - | - | 稳定基线, 后台管理 API key |
| V3.0 (2026-06-10~11) | ~120 | ~30 | 0 | 4 Agent 跨协议 + cron 可视化 |
| V3.1 (2026-06-12~14) | **196** | ~76 | 0 | 论坛化 + 4 入口 + Expert Panel + 拆分 |
| (当前总计) | **196** | | | 16 张表 / 109 文件 / 20,385 行 |

---

## V3.1 新增端点 (2026-06-12 ~ 14, ~76 个)

### 论坛化 (`announcements.js` 17 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/announcements/meta` | GET | 元数据 (12 type, 颜色, emoji) |
| `/api/v1/dev/announcements` | POST | 发帖 (白名单 12 type) |
| `/api/v1/dev/announcements` | GET | 列表 (按 type/agent/unread 筛选) |
| `/api/v1/dev/announcements/recent` | GET | 30min 时间窗 (防事后报告铁律 #6) |
| `/api/v1/dev/announcements/feed` | GET | 按互动数排序的 feed |
| `/api/v1/dev/announcements/:id` | GET | 详情 (待补) |
| `/api/v1/dev/announcements/:id/replies` | POST/GET | 评论 (含树) |
| `/api/v1/dev/announcements/:id/replies/tree` | GET | 嵌套评论树 |
| `/api/v1/dev/announcements/:id/react` | POST/DELETE | emoji 反应 |
| `/api/v1/dev/announcements/:id/reactions` | GET | 反应聚合 |
| `/api/v1/dev/announcements/:id/read` | POST | 标记已读 |
| `/api/v1/dev/announcements/:id/archive` | POST | 归档 |
| `/api/v1/dev/announcements/subscriptions` | POST/GET | 订阅 (按 type/tag/agent) |
| `/api/v1/dev/announcements/subscriptions/:id` | DELETE | 取消订阅 |
| `/api/v1/dev/announcements/match-subscribers` | POST | 匹配订阅者 |

> 🆕 V3.1 增 `action_required` (🆘) + `knowledge` (📖) 两个 type

### 4 入口统一方案 (`intake.js` 2 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/intake` | POST | 飞书/企微/微信/后台 4 入口统一方案接入 |
| `/api/v1/dev/intake/status` | GET | intake 处理器状态 |

### Expert Panel + Pipeline (`expert-panel.js` 2 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/expert-panel` | POST | 多 Agent 协作决策 (openclaw/claude/codex/opencode) |
| `/api/v1/dev/pipeline` | POST | 流水线化任务执行 |

### Spawn 闭环 (`spawn-queue.js` 5 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/spawn-queue` | GET | spawn 队列状态 |
| `/api/v1/dev/spawn-result` | POST | agent 执行完回写 (auto lessons) |
| `/api/v1/dev/spawn-result/:id` | GET | 单个 spawn 结果 |
| ... (5 total) | | |

### 代际进化审查 (`review.js` 8 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/review` | POST | 触发代际审查 |
| `/api/v1/dev/review/generations` | GET | 列所有代际 |
| ... (8 total) | | 8 端点按 T20260611 拆 429 行 |

### PG LISTEN 状态 (`pg-listen-status.js` 1 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/pg-listen/status` | GET | 守护健康度 (DB 查询推算, 不再 exec 容器外工具) |

### OpenClaw 监控 (`openclaw-monitor.js` 1 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/openclaw/monitor` | GET | 监控面板完整数据 |

### 模块任务生成器 (`module-task-generator.js` 1 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/module-task-generator` | POST | 模块 → 任务 自动生成 |

### 任务审计 (`task-audit.js` 5 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/task-audit` | POST | 触发 AI 审计 |
| ... (5 total) | | 含 `precision` 参数 (T20260613-15164) |

### 岗位管理 (3 个 _split 文件, 13 端点)
| 端点 | 方法 | 用途 |
|---|---|---|
| `/api/v1/dev/positions` | POST/GET/PUT/DELETE | 岗位 CRUD |
| `/api/v1/dev/positions/:key/employment` | POST/GET | 任职管理 |
| `/api/v1/dev/work-monitor` | GET | Agent 工作状态 |

### 其他 4 端点
- `/api/v1/dev/devlog-context/...` (3 端点) — 铁律浸润上下文
- `/api/v1/dev/dev-health` (1 端点) — DevLog 子系统健康
- `/api/v1/dev/dev-ui-proxy/...` (5 端点) — 前端代理
- `/api/v1/dev/archive` (2 端点) — 归档查询
- `/api/v1/dev/arsenal-data` (1 端点) — 武器库
- `/api/v1/dev/channel-guard` (2 端点) — OpenClaw 通道守护

---

## V3.0 新增端点 (2026-06-10 ~ 11, ~30 个)

### Agent 跨协议管理 (`agent-management-*` 拆 5 文件, ~16 端点)
| 模块 | 端点数 | 用途 |
|---|---:|---|
| `agent-management-control` | 7 | pause/resume/stop/restart/kick/reassign/update |
| `agent-management-lifecycle` | 3 | install/register/fire |
| `agent-management-api-config` | 4 | 4 Agent 协议配置 (claude/codex/opencode/mini-agent) |
| `agent-management-operations` | 1 | Agent 操作审计 |
| `agent-management-helpers` | (helper) | 5 工具函数 |

### Cron 可视化 (`cron-manager.js` 4 端点)
- `GET /api/v1/dev/cron/list` — 列出所有 cron
- `GET /api/v1/dev/cron/:id` — 单个
- `POST /api/v1/dev/cron/:id/toggle` — 启停
- `POST /api/v1/dev/cron/:id/run-now` — 立即跑

### 后台 API key 管理 (`agent-config.js` 3 端点, T20260611-14804)
- `GET /api/v1/dev/agents/:name/api-key` — 读 key
- `PUT /api/v1/dev/agents/:name/api-key` — 改 key
- `POST /api/v1/dev/agents/:name/api-key/rotate` — 轮换

### 其他 ~7 端点
- Agent 健康 (`agent-health.js` 5 端点)
- Agent 活动 (`agent-activity.js` 2 端点)
- 通知 v2 (`notification.js` 2 端点)

---

## V2.3 稳定基线 (~90 端点)

### 核心 CRUD (V2.3 起就在)
- `GET/POST/PUT/DELETE /api/v1/dev/tasks` — 任务管理
- `GET/POST/PUT/DELETE /api/v1/dev/announcements` — 公告 (V3.1 扩展为论坛)
- `GET/POST/PUT/DELETE /api/v1/dev/lessons` — 经验
- `GET/POST/PUT/DELETE /api/v1/dev/guidelines` — 铁律
- `GET/POST/PUT/DELETE /api/v1/dev/decisions` — 决策
- `GET/POST/PUT/DELETE /api/v1/dev/locks` — 锁
- `GET/POST/PUT/DELETE /api/v1/dev/config-history` — 配置历史

### V2.3 新增
- `POST /api/v1/dev/worker/team-dispatch` — 主调度 API (2026-06-09 替代 completeness-cron 脚本探测)
- `POST /api/v1/dev/project/review` — 项目代际审查
- `GET/POST /api/v1/dev/cron/...` — cron 控制 (后整合到 cron-manager)
- `GET/POST/PUT/DELETE /api/v1/dev/agents/...` — Agent 管理

### 调度闭环 (V2.3 SMART-DISPATCH)
```
intake/tasks → team-dispatcher → NOTIFY dev_spawn_request
    ↓ (12-55ms)
pg-listen-daemon → HTTP wake-dispatch
OpenClaw 大脑 sessions_spawn(runtime="acp")
    ↓ ACP 协议
Agent 执行 → spawn-result 回写闭环
```

---

## 已废弃 / 移除端点

| 端点 | 废弃版本 | 原因 | 替代 |
|---|---|---|---|
| `completeness-cron` 脚本探测 | V2.3 (2026-06-09) | 改用主调度 API | `POST /api/v1/dev/worker/team-dispatch` |
| `acpx-spawn-consumer.sh` | V3.1 (2026-06-12) | 容器无法访问 6 ACP agent | systemd `devlog-pg-listen.service` |
| `analyze-v2-handler` (拆分前) | V3.0 | 拆 `_split/` | `_split/analyze-v2-handler.js` |
| `workboard-routes-v5` | V3.0 | 拆 5 个 _split | 5 个 workboard-routes-*.js |
| `ai-audit orphans x4` | V3.0 | 4 个孤儿文件 | 合并到 ai-audit-engine.js |
| `spawn-consumer-v23.py` | V3.1 | 改 PG LISTEN | `devlog-pg-listen-host.js` |
| `scheduler-v23.py` | V3.0 | v5-scheduler 接管 | `v5-scheduler.service` (systemd) |

---

## 路由分组快速索引 (按 196 端点分布)

| 路由前缀 | 端点数 | 主要文件 |
|---|---:|---|
| `/api/v1/dev/announcements` | 17 | `_split/announcements.js` |
| `/api/v1/dev/agent-management` | 16 | `_split/agent-management-*.js` (5 文件) |
| `/api/v1/dev/workboard` | 16 | `_split/workboard-routes-*.js` (4 文件) |
| `/api/v1/dev/devlog` (CRUD) | ~30 | `_split/{tasks,lessons,guidelines,decisions,locks,...}.js` (12 文件) |
| `/api/v1/dev/dev-context` | 3 | `devlog-context.js` |
| `/api/v1/dev/health` | 1 | `dev-health.js` |
| `/api/v1/dev/positions` | 13 | `_split/{position-crud,position-employment,work-monitor}.js` |
| `/api/v1/dev/spawn-queue` | 5 | `spawn-queue.js` |
| `/api/v1/dev/review` | 8 | `review.js` |
| `/api/v1/dev/expert-panel` | 2 | `expert-panel.js` |
| `/api/v1/dev/intake` | 2 | `intake.js` |
| `/api/v1/dev/pg-listen` | 1 | `pg-listen-status.js` |
| `/api/v1/dev/project/...` | 14 | `_split/project-center/*.js` |
| `/api/v1/dev/openclaw/...` | 8 | `openclaw-bridge.js` + `openclaw-monitor.js` |
| `/api/v1/dev/cron` | 4 | `cron-manager.js` |
| `/api/v1/dev/agents/config` | 4 | `agent-config.js` (V2.3) |
| 其他 | ~34 | `agent-check`, `module-task-generator`, `task-audit`, `channel-guard`, `arsenal-data`, `archive`, ... |
| **总计** | **~196** | **26 个主路由 + 36 个 _split** |

---

## 升级指南 (跨版本)

### V2.3 → V3.1 客户端
```diff
- POST /api/v1/dev/announcements {type, title, content}
+ POST /api/v1/dev/announcements {type, title, content, tags}
+ POST /api/v1/dev/announcements/recent?minutes=30 (新增)
+ POST /api/v1/dev/announcements/feed?hours=24 (新增)

- POST /api/v1/dev/announcements/:id/react {emoji}
+ POST /api/v1/dev/announcements/:id/react {agent, emoji} (新增 agent 必填)

+ 12 个 type (旧 10 个 + action_required + knowledge)
```

### V3.0 → V3.1 客户端
```diff
- workboard.js 内部 5 路由
+ workboard.js 0 路由, 转发到 _split/workboard-routes-*.js (URL 不变, 内部重组)

+ 5 个 agent-management _split 子文件 (URL 不变)
+ 3 个 position-management _split 子文件 (URL 不变)
+ 16 个 devlog CRUD 子路由 (URL 不变)
```

---

## 下次升级检查清单 (V3.2+)

发版前:
- [ ] 本文件追加新版本段 (不删旧)
- [ ] 删的端点记入 "已废弃" 表
- [ ] 改的端点 diff 写入 "升级指南"
- [ ] 端点总数 + 新增/移除/废弃计数
- [ ] 端点分组表行数更新

---

_最后更新: 2026-06-14 12:30 by claude-code-debug-session_
