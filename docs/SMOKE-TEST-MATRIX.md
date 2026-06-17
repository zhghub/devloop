# DevLog 196 端点 Smoke Test 矩阵

> 创建: 2026-06-14 | 范围: /api/v1/dev/* 所有端点 (196 个) + 相关 /api/v1/* 子集
> 策略: 现有 verify-*.sh 是手写 smoke test, 缺系统化矩阵. 本文档给完整覆盖地图
> 状态: 设计阶段, P0 端点给实际脚本, P1/P2 给模板和待办

## 优先级定义

| P | 含义 | 测试深度 |
|---|---|---|
| **P0** | 业务关键, 出错 = 业务不可用 | 真实数据, 验证 response schema + DB 写入 + 副作用 |
| **P1** | 重要, 出错 = 部分功能受影响 | 真实数据, 验证 status code + 关键字段 |
| **P2** | 辅助, 出错 = 体验降级 | dryRun 模式 + 验证可达 + 关键字段存在 |

## 已存在的 verify 脚本 (8 个, 全部 P0)

| 脚本 | 覆盖端点 | 频率 |
|---|---|---|
| `verify-acp-alert.sh` | `/acp-alert/*` (4 端点) | 手动 |
| `verify-agent-dispatch-fallback.sh` | `/team-dispatch` fallback | 手动 |
| `verify-c-test-fallback-chain.sh` | C test fallback | 手动 |
| `verify-devlog-contract.sh` | contract-guard 验证 | 手动 |
| `e2e-host-verify.sh` | 多端点 e2e | 手动 |
| `headless-verify.mjs` | headless 端点 | 手动 |
| `test-harness-routing.sh` | routing 测试 | 手动 |
| `run-tests.py` | 通用 runner | 手动 |

**未覆盖**: ~150+ 端点 (主要是 _split/ 子模块)

---

## 完整 196 端点矩阵 (按 P 级 + 域分组)

### 🔴 P0 - 必测 (业务关键, ~35 端点)

| 域 | 端点 | 方法 | 测试目标 | 现有 |
|---|---|---|---|---|
| **announcements** | `/announcements` | POST | 创建论坛帖, 验证 12 type 白名单 | ❌ |
| | `/announcements/:id/replies` | POST | 评论, 验证树形结构 | ❌ |
| | `/announcements/:id/react` | POST | emoji 反应, 验证 upsert | ❌ |
| | `/announcements/feed` | GET | feed 排序按互动数 | ❌ |
| | `/announcements/recent?minutes=30` | GET | 30min 时间窗 (铁律 #6) | ❌ |
| **spawn-queue** | `/spawn-result` | POST | agent 回写 (auto lessons 触发) | ❌ |
| | `/spawn-queue` | GET | 队列状态 | ❌ |
| **team-dispatch** | `/worker/team-dispatch` | POST | 主调度, 验证 4-agent 派发 | ❌ |
| **devlog-context** | `/context/summary` | GET | 铁律浸润上下文 | ❌ |
| | `/context/prompt` | GET/POST | 同上 | ❌ |
| **tasks** | `/tasks` | GET/POST/PUT | 任务 CRUD + FP 拦截 | ❌ |
| | `/tasks/:id/...` | 各种 | 状态机 + spawn 闭环 | ❌ |
| **review** | `/review` | POST | 代际进化审查 | ❌ |
| | `/review/generations` | GET | 列所有代际 | ❌ |
| **intake** | `/intake` | POST | 4 入口统一方案 | ❌ |
| | `/intake/status` | GET | 处理器状态 | ❌ |
| **expert-panel** | `/expert-panel` | POST | 多 Agent 协作 | ❌ |
| | `/pipeline` | POST | 流水线 | ❌ |
| **agent-management-control** | pause/resume/stop | POST | Agent 控制 (7 端点) | ❌ |
| **agent-management-lifecycle** | install/register/fire | POST | Agent 生命周期 (3) | ❌ |
| **agent-management-api-config** | `:name/api-key` | GET/PUT | API key 管理 (4) | ❌ |
| **dev-health** | `/health` | GET | overall=ok + 真实数字 (修后) | ❌ |
| **pg-listen-status** | `/pg-listen/status` | GET | 宿主机 systemd 状态 | ❌ |
| **cron-manager** | `/cron/list` | GET | cron 列表 (V2.3) | ❌ |
| | `/cron/:id/toggle` | POST | 启停 | ❌ |

### 🟡 P1 - 应测 (重要, ~80 端点)

| 域 | 端点 | 测什么 |
|---|---|---|
| **workboard-routes-aggregator** | 7 端点 | 10 卡片汇总, 数据完整性 |
| **workboard-routes-intelligence** | 4 端点 | 雷达图 + 健康分 |
| **workboard-routes-resources** | 3 端点 | CPU/内存/磁盘读数 |
| **workboard-routes-misc** | 2 端点 | 任务分布 + 趋势 |
| **dashboard-radar** | 6 端点 | 模块/任务健康度 |
| **announcements 论坛** | `/subscriptions` 系列 (5 端点) | 订阅匹配 |
| | `/archive` `/read` | 已读 + 归档 |
| **agent-health** | 5 端点 | 实时 agent 健康 |
| **agent-registry** | 2 端点 | Agent 注册 CRUD |
| **agent-activity** | 2 端点 | 活动日志 |
| **announcements/_split** | 12 端点 (其余) | meta, tree, single, recent |
| **notifications** | 2 端点 | 飞书/邮件通知 |
| **lessons / lessons-extra** | 7 端点 | 教训 CRUD + 详情 |
| **guidelines** | 5 端点 | 铁律 CRUD |
| **decisions** | 2 端点 | 决策记录 |
| **locks** | 4 端点 | 锁 acquire/release |
| **quality** | 2 端点 | 质量检查 |
| **task-audit** | 5 端点 | AI 审计 + precision 参数 |
| **stats / summary** | 2 端点 | 统计 + 摘要 |
| **config-history** | 2 端点 | 配置变更 |
| **announcements 详情** | `/:id`, `/meta` | 详情 + 元数据 |
| **channel-guard** | 2 端点 | OpenClaw 通道守护 |
| **position-crud** | 5 端点 | 岗位 CRUD |
| **position-employment** | 5 端点 | 任职管理 |
| **work-monitor** | 3 端点 | Agent 工作状态 |
| **archive** | 2 端点 | 归档查询 |
| **export-import** | 2 端点 | 数据导出导入 |
| **arsenal-data** | 1 端点 | 武器库 |
| **module-task-generator** | 1 端点 | 模块→任务自动生成 |
| **dev-ui-proxy** | 5 端点 | 前端代理 |
| **openclaw-bridge** | 7 端点 | OpenClaw 桥接 |
| **openclaw-monitor** | 1 端点 | 监控面板 |

### 🟢 P2 - 选测 (辅助, ~80 端点)

| 域 | 端点 | 测什么 |
|---|---|---|
| **project-center** | 14 端点 (`/project/*`) | 项目指挥中心 |
| **dashboard** | 1 端点 | 仪表盘快照 |
| **agent-check** | 1 端点 | truthy 检测 |
| **agent-config** | 3 端点 | V2.3 API key (3 端点) |
| **analyze-v2** | 3 端点 | 方案解析 |

---

## 测试模板 (3 个通用 pattern)

### 模板 A: GET 列表类
```bash
# 1. 状态码
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$URL" -H "$AUTH")
[[ "$HTTP_CODE" == "200" ]] || { echo "FAIL: expected 200 got $HTTP_CODE"; exit 1; }

# 2. 响应 schema (用 jq 验证字段)
RESP=$(curl -s "$URL" -H "$AUTH")
echo "$RESP" | jq -e '.success == true' >/dev/null || { echo "FAIL: success not true"; exit 1; }
echo "$RESP" | jq -e '.data | type == "array"' >/dev/null || { echo "FAIL: data not array"; exit 1; }

# 3. 数据合理性 (至少有 1 条 / 不超 limit)
COUNT=$(echo "$RESP" | jq '.data | length')
[[ "$COUNT" -ge 1 ]] || { echo "FAIL: empty data"; exit 1; }
[[ "$COUNT" -le 20 ]] || { echo "FAIL: too many $COUNT"; exit 1; }
```

### 模板 B: POST 写后读
```bash
# 1. 准备测试数据
TITLE="smoke-test-$(date +%s)"
CREATE=$(curl -s -X POST "$BASE/announcements" -H "$AUTH" -H "Content-Type: application/json" -d "{
  \"type\":\"info\",
  \"title\":\"$TITLE\",
  \"content\":\"smoke test body\",
  \"created_by\":\"smoke-test\"
}")
ID=$(echo "$CREATE" | jq -r '.data.id')
[[ "$ID" != "null" && -n "$ID" ]] || { echo "FAIL: no id"; exit 1; }

# 2. 读回
READ=$(curl -s "$BASE/announcements/$ID" -H "$AUTH" 2>/dev/null || curl -s "$BASE/announcements?created_by=smoke-test&limit=1" -H "$AUTH")
echo "$READ" | jq -e ".data[0].title == \"$TITLE\"" >/dev/null || { echo "FAIL: not in list"; exit 1; }

# 3. 清理
curl -s -X POST "$BASE/announcements/$ID/archive" -H "$AUTH" >/dev/null
```

### 模板 C: 状态机/权限
```bash
# 1. 未鉴权 → 401
NO_AUTH=$(curl -s -o /dev/null -w "%{http_code}" "$URL")
[[ "$NO_AUTH" == "401" ]] || { echo "FAIL: should 401 unauth"; exit 1; }

# 2. 错误鉴权 → 401
BAD_AUTH=$(curl -s -o /dev/null -w "%{http_code}" "$URL" -H "x-internal-secret: WRONG")
[[ "$BAD_AUTH" == "401" ]] || { echo "FAIL: should 401 bad auth"; exit 1; }

# 3. 正确鉴权 → 200
GOOD=$(curl -s -o /dev/null -w "%{http_code}" "$URL" -H "x-internal-secret: $INTERNAL")
[[ "$GOOD" == "200" ]] || { echo "FAIL: should 200 good auth"; exit 1; }

# 4. POST 业务字段验证 (例如 type 白名单)
INVALID_TYPE=$(curl -s -X POST "$URL" -H "$AUTH" -d '{"type":"NOT_REAL"}' -H "Content-Type: application/json" | jq -r '.data.type')
[[ "$INVALID_TYPE" == "info" ]] || { echo "FAIL: invalid type not downgraded"; exit 1; }
```

---

## 实施路线图

### Phase 1 (P0, 本周, 35 端点)
- [ ] `smoke-test-announcements.sh` — 12 端点 (核心论坛)
- [ ] `smoke-test-spawn.sh` — 5 端点 (spawn 闭环)
- [ ] `smoke-test-dispatch.sh` — 2 端点 (主调度)
- [ ] `smoke-test-context.sh` — 3 端点 (铁律浸润)
- [ ] `smoke-test-tasks.sh` — 5 端点 (任务 CRUD)
- [ ] `smoke-test-review.sh` — 3 端点 (代际审查)
- [ ] `smoke-test-intake-expert.sh` — 4 端点 (入口 + 协作)
- [ ] `smoke-test-agent-control.sh` — 14 端点 (agent 管理 5 文件)
- [ ] `smoke-test-health.sh` — 2 端点 (健康 + pg-listen)

**小计**: 9 个脚本, 35 端点 (1 脚本/域)

### Phase 2 (P1, 2 周, 80 端点)
- [ ] `smoke-test-workboard.sh` — 16 端点 (workboard 4 文件)
- [ ] `smoke-test-dashboard.sh` — 6 端点
- [ ] `smoke-test-lessons-guidelines.sh` — 12 端点
- [ ] `smoke-test-agent-mgmt.sh` — 9 端点
- [ ] `smoke-test-position.sh` — 13 端点
- [ ] `smoke-test-misc.sh` — 18 端点 (其他 P1)

### Phase 3 (P2, 1 月, 80 端点)
- [ ] `smoke-test-project-center.sh` — 14 端点
- [ ] `smoke-test-dashboard-snapshot.sh` — 1 端点
- [ ] 补 agent-config / analyze-v2 / agent-check

### Phase 4 (持续)
- [ ] 改 `node:test` 或 `pytest` (Python 镜像) 替换 bash, 拿到结构化报告
- [ ] 加 CI: 每次 PR 跑全套 P0, merge 前 P0 全绿
- [ ] 加覆盖率报告 (实际跑通端点 / 总端点)

---

## 验收标准

✅ 全部 P0 端点 100% 覆盖  
✅ 端点 schema 变化时 (P0 + 涉及列) 测试自动失败  
✅ 测试用真实数据, 不 mock (发现真问题比假绿有用)  
✅ 测试 < 5 分钟跑完 (CI 友好)  
✅ 测试产物 (response 样本) 存 /var/log/smoke-test/ 备查  

## 与现有 doc-maintain cron 的协同

- doc-maintain 检查文档, smoke-test 检查行为
- 都发 alert 到 devlog 论坛 (失败时不静默)
- 都 1 天 1 alert 防抖

---

_最后更新: 2026-06-14 12:35 by claude-code-debug-session_
