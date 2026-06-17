# 📐 Design Decisions (ADR)

> Architecture Decision Records — DevLoop 的关键设计决策。
> 格式: Context (背景) / Decision (决策) / Consequences (后果)

---

## ADR-001: 为什么选 ACP 协议,不用 OpenAI Function Calling?

**日期**: 2026-06-11
**状态**: ✅ 已采纳

### Context (背景)

Agent 跟 LLM 通信有 2 大主流协议:
- OpenAI Function Calling (业界事实标准)
- ACP (Agent Client Protocol, Zed 团队, 对标 LSP)

DevLoop 想支持 12 个 AI 编码工具 (Claude/Codex/Gemini/...)。

### Decision (决策)

**采用 ACP 协议**,不走 OpenAI Function Calling。

### Consequences (后果)

✅ **Vendor Neutral** — 12 个 harness 都能接,不锁单一 LLM
✅ **生命周期管理** — ACP 内置 session/timeout/reconnect
✅ **细粒度权限** — permissionMode (strict/relaxed/approve-all)
✅ **未来扩展** — ACP 0.6.1 稳定, Zed 团队持续维护

❌ **学习曲线** — 团队需要理解 JSON-RPC over stdio
❌ **社区生态小** — 比 Function Calling 文档少

---

## ADR-002: 为什么不用 Prisma,用原生 pg?

**日期**: 2026-06-10
**状态**: ✅ 已采纳

### Context (背景)

主流 Node.js 项目用 Prisma 做 ORM。DevLoop 模块也可以用。

### Decision (决策)

**用原生 `pg` (node-postgres) + `pool.query()`**,不用 Prisma client。

### Consequences (后果)

✅ **零抽象开销** — SQL 直接,无 ORM 学习成本
✅ **复杂查询友好** — V3.1 5 维审查的 JSONB / 自定义函数裸写更顺
✅ **剥离友好** — 独立 devloop 仓库只需复制 SQL migrations
✅ **启动快** — 无 Prisma client 生成步骤

❌ **类型安全弱** — 没有自动生成的 TypeScript 类型
❌ **手写 SQL** — 复杂 JOIN / 嵌套查询要小心

---

## ADR-003: 为什么数据底座 + 智能体分层?

**日期**: 2026-06-09
**状态**: ✅ 已采纳

### Context (背景)

DevLoop 是 fastify plugin 模式,跟主 backend (小辉 CRM/ERP) 共享 fastify 实例。但智能体调度 (openclaw 大脑) 跟数据持久化职责不同。

### Decision (决策)

**两层分工**:
- **数据底座** = DevLoop (Fastify + PostgreSQL, 持久化 + 检索, 不做决策)
- **智能体层** = OpenClaw 大脑 + ACP Agent (调度 + 决策 + 执行)

### Consequences (后果)

✅ **职责清晰** — 数据库不"思考", 大脑不"存储"
✅ **可测试** — 数据底座可独立 mock,智能体可独立 dry-run
✅ **可替换** — 大脑可以从 openclaw 换成 claude-main
✅ **可观测** — 数据底座 196 API 全开放,智能体行为有 audit log

❌ **两层通信** — 大脑 → 数据底座 → Agent 多一跳
❌ **数据一致性** — 状态同步要小心

---

## ADR-004: 为什么 8 角色设计,不是单一 agent?

**日期**: 2026-06-12
**状态**: ✅ 已采纳 (V3.1 终态)

### Context (背景)

传统 agent 框架 (AutoGen/CrewAI) 强调"多 agent",但角色设计随意。DevLoop 要解决"项目协作"。

### Decision (决策)

**8 角色固定**,职责分明:
- leader (调度) / intake (接入) / plan-parsing (解析)
- reviewer (审查) / developer (开发) / auditor (审计)
- assistant (辅助) / doc (复盘)

### Consequences (后果)

✅ **职责无重叠** — 每个任务有明确归属
✅ **可观测** — 任务 → 角色 → Agent 一一对应
✅ **易扩展** — 加新角色不影响现有
✅ **代际进化清晰** — 每代审查归属于 reviewer 角色

❌ **角色转换** — 跨角色任务要协调
❌ **资源浪费** — 简单任务不需要 8 角色

---

## ADR-005: 为什么代际进化审查,不用一次性 LLM 评估?

**日期**: 2026-06-12
**状态**: ✅ 已采纳 (V3.1 重点)

### Context (背景)

传统 AI 代码审查 (CodeRabbit/Sourcery) 都是一次性 LLM 调用,每次审查独立,不会学习。

### Decision (决策)

**代际进化审查**:
- 第 N 代审查读 N-1 代审查数据 (findings, quality_score, task_completion_rate)
- 动态调整本轮审查策略 (深挖 / 细化 / 严判)
- 质量分公式: `score = weighted(finding_count, severity, task_completion_rate)`

### Consequences (后果)

✅ **越用越准** — 第 N+1 代审查比 N 代更精准
✅ **减少误报** — 任务完成率反馈调整审查阈值
✅ **自适应** — 不需要人工调参

❌ **数据需求** — 需要至少 5+ 代数据才有意义
❌ **冷启动慢** — 第 1 代审查跟传统一样

---

## ADR-006: 为什么 196 REST API 全开放?

**日期**: 2026-06-10
**状态**: ✅ 已采纳

### Context (背景)

多数 agent 框架 (LangChain/CrewAI) 是 SDK 嵌入式,用户想看数据 / 改行为只能改源码。

### Decision (决策)

**196 个 REST API 全部开放**:
- 所有数据可查 (tasks/lessons/guidelines/decisions/...)
- 所有行为可控 (pause/resume/restart/kick agents)
- 所有审计可读 (audit-runs/alert-history)

### Consequences (后果)

✅ **可观测** — 任何状态都可通过 API 查询
✅ **可集成** — 第三方工具可接入
✅ **可审计** — 满足合规需求
✅ **教学友好** — 学习者可逐步探索

❌ **API 维护成本** — 196 端点要长期兼容
❌ **文档压力** — 必须有完整 API 文档

---

## ADR-007: 为什么剥离 devlog,从 xiaohui-2.0-dev 单独开源?

**日期**: 2026-06-14
**状态**: ✅ 已采纳

### Context (背景)

DevLoop 模块嵌入小辉数字员工 2.0 系统的 fastify plugin 里,跟 CRM/ERP 业务代码混在一起。

### Decision (决策)

**剥离成独立产品 `devloop`**:
- 跟业务 0 耦合 (已验证)
- MIT 开源
- 单独 docker compose 部署
- 域名 `devloop.cn`, GitHub `zhghub/devloop`

### Consequences (后果)

✅ **独立发展** — 不被小辉业务节奏绑架
✅ **开源潜力** — 任何团队可独立使用
✅ **社区建设** — 不限于"小辉用户"
✅ **商业模式清晰** — Open core (核心开源 + 高级功能收费)

❌ **剥离工作量** — 20h 起步
❌ **同步成本** — 小辉业务更新要同步合并

---

## ADR-008: 为什么不用 monorepo,独立仓库?

**日期**: 2026-06-15
**状态**: ✅ 已采纳

### Context (背景)

现代开源项目常用 monorepo (Nx/Turborepo) 管理多个 package。

### Decision (决策)

**独立仓库** — `zhghub/devloop`,不嵌进 monorepo。

### Consequences (后果)

✅ **仓库轻量** — clone 速度快
✅ **CI 简单** — 一个 workflow 就够
✅ **Issue 管理** — 不用 monorepo 标签过滤

❌ **多仓协调** — 后续 devloop + 小辉业务要双向同步
❌ **依赖共享** — 重复 node_modules (但 docker 镜像复用)

---

## 📋 ADR 清单

| ID | 标题 | 日期 | 状态 |
|---|---|---|---|
| 001 | ACP vs OpenAI Function Calling | 2026-06-11 | ✅ |
| 002 | 原生 pg vs Prisma | 2026-06-10 | ✅ |
| 003 | 数据底座 + 智能体分层 | 2026-06-09 | ✅ |
| 004 | 8 角色设计 | 2026-06-12 | ✅ |
| 005 | 代际进化审查 | 2026-06-12 | ✅ |
| 006 | 196 REST API 全开放 | 2026-06-10 | ✅ |
| 007 | 剥离 devlog 独立开源 | 2026-06-14 | ✅ |
| 008 | 独立仓库 vs monorepo | 2026-06-15 | ✅ |

---

🔄 [devloop.cn](https://devloop.cn) · [GitHub](https://github.com/zhghub/devloop)
