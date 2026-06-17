# DevLoop V3.1 源码地图 (MAP)

> 最后更新: 2026-06-12 15:50 | 配套: DEVLoop-WORKFLOW-V3.md | 源码版本: V3.1

## 一、系统定位

DevLog = 小辉数字员工2.0 AI开发任务调度中枢。

**调度闭环 (V3.1 final)**:
```
intake/tasks → team-dispatcher → NOTIFY dev_spawn_request
    ↓ (实时, 12-55ms)
pg-listen-daemon (进程内 PG LISTEN, 替代 acpx-spawn-consumer.sh)
    ↓ HTTP wake-dispatch
OpenClaw 大脑 sessions_spawn(runtime="acp")
    ↓ ACP 协议
Agent 执行 → spawn-result 回写闭环
```
0外部脚本，全 ACP 协议通信。

可用Agent: openclaw(leader) / claude-main(developer) / codex(reviewer) / opencode(auditor+assistant)

## 二、源码总览 (85 源文件, ~15,000 行)

| 目录 | 文件数 | 说明 |
|------|:---:|------|
| routes/ | 21 | API端点, ~16个主路由 + 5个 _split/ 子目录 |
| routes/_split/ | 31 | 拆分的子路由 (tasks/announcements/workboard/agent-mgmt/project-center) |
| services/ | 13 | 核心业务逻辑 (调度/审计/通知/密钥/PG LISTEN守护) |
| services/agent-writers/ | 5 | 4Agent 跨协议配置写入 |
| services/_split/ai-audit/ | 7 | AI审计引擎 (LLM调用/判决/进化) |
| services/_split/team-dispatcher/ | 4 | 🆕 调度核心拆分 (pipeline/roles/spawn/validation) |
| scripts/ | 1 | CLI工具 (devlog.sh, AGENTS.md 推荐路径) |

##三、关键文件行数 (agent扫描验证)

| 文件 | 行数 | 说明 |
|------|------|------|
| routes/_split/announcements.js | 525 | 公告+论坛 (最大单文件) |
| routes/_split/workboard-cards.js | 441 | 10卡片数据层 |
| routes/dashboard-radar.js | 439 | 雷达图+健康分 |
| routes/review.js | 429 | 🆕 代际进化审查 |
| routes/_split/agent-management-control.js | 429 | Agent控制 |
| routes/team-dispatch.js | 424 | 核心调度API |
| routes/_split/tasks.js | 367 | 任务CRUD+FP拦截 |
| routes/analyze-v2.js | 365 | 方案解析V2 |
| routes/_split/project-center/common/versions.js | 343 | 版本CRUD |
| services/agent-registry-truthy.js | 325 | 真活检测 |
| services/guideline-registry.js | 324 | 铁律注册中心 |
| services/team-dispatcher.js | 302 | 🆕 调度facade (669→298→302) |
| services/devlog-contract-guard.js | 297 | 4层硬约束 |
| services/auto-review-cron.js | 286 | 自动审查3层 |
| routes/agent-health.js | 285 | Agent健康 |
| services/agent-coordinator.js | 259 | Agent协调器 |
| services/perf-monitor.js | 258 | LRU缓存+性能 |
| routes/openclaw-bridge.js | 248 | OpenClaw桥接 |
| routes/spawn-queue.js | 234 | 🆕 spawn-result+auto lessons |
| routes/intake.js | 227 | 🆕 4入口统一方案 |
| services/devlog-notification-service.js | 219 | 飞书通知 |
| routes/expert-panel.js | 201 | 🆕 Expert Panel+Pipeline |
| routes/devlog-context.js | 188 | 铁律浸润上下文 |
| services/_split/team-dispatcher/spawn.js | 182 | spawn逻辑 |
| routes/_split/lessons-extra.js | 172 | 教训详情 |
| services/_split/team-dispatcher/roles.js | 172 | 角色映射 |
| services/agent-writers/codex.js | 158 | Codex TOML写入 |
| routes/cron-manager.js | 157 | Cron管理 |
| routes/_split/workboard-routes-resources.js | 134 | 资源消耗 |
| routes/_split/guidelines.js | 131 | 铁律CRUD |
| services/_split/team-dispatcher/validation.js | 131 | FP预检+4段 |
| routes/_split/project-center/common/dashboard.js | 130 | 项目快照 |

## 四、V3.1 新增 (10文件)

| 文件 | 行数 |
|------|------|
| routes/intake.js | 227 |
| routes/expert-panel.js | 201 |
| services/_split/team-dispatcher/pipeline.js | 77 |
| services/_split/team-dispatcher/roles.js | 172 |
| services/_split/team-dispatcher/spawn.js | 182 |
| services/_split/team-dispatcher/validation.js | 131 |
| services/devlog-pg-listen-daemon.js | 🆕 195 | PG LISTEN 守护 (替代 acpx-spawn-consumer.sh) |
| routes/pg-listen-status.js | 🆕 20 | 守护状态端点 |
| migrations/20260611_devlog_v3_acpx_session.sql | 9 |
| scripts/report-task-helper.sh | 101 |

## 五、V3.1 重构 (5文件)

| 文件 | 旧→新行数 |
|------|------|
| services/team-dispatcher.js | 669→302 (facade + 4split) |
| routes/review.js | ~250→429 (+代际) |
| routes/spawn-queue.js | 111→234 (+spawn-result) |
| routes/agent-check.js | 19→45 (truthy.probeAgent) |
| services/devlog-contract-guard.js | 297→330 (识别 pg-listen 新consumer) |

## 六、已删除 (9项)

workboard-routes-v5.js / analyze-v2-handler.js / ai-audit orphans x4 / spawn-consumer-v23.py / scheduler-v23.py / acpx-spawn-consumer.sh (废弃 → pg-listen-daemon取代)

> **2026-06-15 补充 (海刚 16:55 清理)**: 本节扩到 18 项.
> - `scripts/devlog-dispatch.py` / `scripts/devlog-audit-deepseek.py` (6/11 25fe442d V3.0 铲除)
> - `scripts/spawn-consumer-s1-check.sh` / `scripts/devlog-team-dispatch-cron.sh` / `scripts/minimax-heartbeat.sh` (6/15 17:05 STOPPED-0615-1557)
> - `scripts/devlog-auto-dispatch.sh` (6/15 17:22 V3.1 修订文档 #10)
> - `backend/src/modules/devlog/scripts/devlog.sh` (6/15 17:22 重复)
> - `backend/src/modules/devlog/services/_split/s1-checker.js` (6/15 17:30 V23 S1 自检, 后端 0 引用)
> - `skills/devlog-dispatch/devlog-dispatch` (6/15 17:05 死链 symlink)
> - `backend/coverage/` 60MB / 2094 文件 (6/15 17:05 .gitignore 加规则)
>
> V3.1 范式下唯一 dispatcher: 宿主机 systemd `devlog-pg-listen.service` (300 行 `devlog-pg-listen-host.js`).

## 七、API (~90端点)

新增V3.1: POST /spawn-result, /expert-panel, /pipeline, /intake | GET /intake/status, /pg-listen/status, /agents/keys, /agents/writers/capabilities

## 八、数据库 (13表, V3.1 +2列)

dev_task_logs +acpx_session_id | dev_spawn_queue +acpx_session_id +session_status

## 九、配置变更

~/.acpx/config.json(Claude IS_SANDBOX=1+opencode) | .acpxrc.json(approve-all) | docker-compose.yml(+/host-nvm) | OpenClaw(permissionMode=approve-all)

## 十、P2债务: 3/6已修, 3低风险残留

✅ INJECTION_PATTERNS去重 ✅ crypto seed警告 ✅ markInProgressAndLock日志 ⚠️ TOML脆弱 ⚠️ fallback.jsonl重复 ⚠️ 低风险

## 十一、今日commit (2026-06-12)

666f347 intake | 3f2c1cb 代际审查 | 0d9a6e4 一致性+auto | 2105370 5cron钩子 | 5dbefca5 P2债务+agent对齐

---

_基于全源码重读 + agent扫描验证, 2026-06-12_
