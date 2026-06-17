# 🔄 DevLoop

> **你的 AI 写代码写完了 — 然后呢?**

---

你的 AI 写完代码 → 然后呢?

谁来审查?谁来复盘?谁来保证下次写得更好?

**DevLoop 给你的不是一个 agent,而是一支团队。**

- 🧠 **leader** 把控节奏,**reviewer** 审查代码
- 🛠️ **developer** 写代码,**auditor** 做安全扫描
- 🔁 审查每轮都比上次更准 (代际进化)
- 📋 任务自动派,完成自动写 lessons
- 🔌 Claude / Codex / Gemini ... 12 个 AI 工具都能上

围绕你的 Git 项目,本地 + 异地 Agent 协同干活。
**196 REST API 全开放**,不是黑盒,你自己说了算。

---

## ⚡ 30 秒上手

```bash
git clone https://github.com/zhghub/devloop
cd devloop
docker compose up -d

# 验证
curl http://localhost:3001/api/v1/dev/health
# {"status":"ok","agents_alive":4}
```

---

## 🎯 为什么是 DevLoop?

| 痛点 | 别人怎么做 | DevLoop 怎么做 |
|---|---|---|
| AI 写完代码没人审查 | LangChain / CrewAI: 你自己接 SDK | 8 角色内置,reviewer 自动审查 |
| 审查每次都要从头来 | Aider: 单 agent 自己干 | 代际审查,质量分自动提升 |
| 被某家 LLM 锁死 | MetaGPT: 只支持自家模型 | 12 个 ACP harness 全兼容 |
| 装上发现是"假活" | 多数框架: 装上就算能用 | DB + CLI 双重探活 |
| 想看数据改不动 | 多数项目: 黑盒 SDK | 196 REST API 全开放 |

---

## 🔌 兼容 12 个 ACP Harness

任何支持 [ACP 协议](https://agentclientprotocol.com) 的 AI 编码工具都能接入:

`Claude` · `Codex` · `Copilot` · `Cursor` · `Droid` · `Gemini` ·
`iflow` · `Kilocode` · `Kimi` · `Kiro` · `OpenCode` · `Qwen`

切换 LLM 不用改代码,改配置就行。

---

## 🏗️ 架构

```
┌─────────────────────────────────────────┐
│         你的 Git 项目                     │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  DevLoop 数据底座 (Fastify + PostgreSQL) │
│  任务总线 · 经验库 · 铁律 · 论坛 · 仪表盘  │
│  16 张表 / 196 REST API / 全开放          │
└──────────────────┬──────────────────────┘
                   ↓ ACP 协议
┌─────────────────────────────────────────┐
│     AI Agent 团队 (8 角色)               │
│  leader · developer · reviewer · auditor │
│  intake · architect · assistant · doc    │
└─────────────────────────────────────────┘
```

---

## 📚 文档

- [V3.1 工作流 (权威)](docs/DEVLOG-WORKFLOW-V3.md)
- [V2.3 系统方案](docs/DEVLOG-V2.3-MASTER.md)
- [总指挥手册](docs/SMART-DISPATCH-V2.md)
- [ACP Harness 矩阵](docs/ACP-AGENT-CAPABILITY-MATRIX.md)
- [路线图](docs/ROADMAP.md) · [架构概览](docs/ARCHITECTURE.md) · [设计决策](docs/DESIGN_DECISIONS.md)
- [API 变更日志](docs/API-CHANGELOG.md) · [数据库 ER 图](docs/devlog-ER.md) · [测试矩阵](docs/SMOKE-TEST-MATRIX.md)

---

## 🌳 DevLoop 来自哪里?

DevLoop 是从 **小辉数字员工 2.0** (AI 企业 ERP/CRM 系统) 里**剥离出来的开源模块**。

```
┌─────────────────────────────────────────────┐
│         小辉数字员工 2.0 (母系统)            │
│   22 后端模块 · 152 张表 · 飞书/企微/微信集成  │
│                                             │
│   ┌──────────────────────────────────────┐ │
│   │ DevLog (开发任务调度中枢 · 8 角色)    │ │
│   │     👉  剥离 → 开源 = DevLoop        │ │
│   └──────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

**DevLoop 拿走的**: 数据底座 + 智能体调度 (0 业务耦合)
**DevLoop 留下的**: 跟小辉 CRM/ERP/商城 完全无关

📚 小辉数字员工系统方案文档: [`docs/xiaohui/`](docs/xiaohui/)
- [小辉数字员工系统总体方案](docs/xiaohui/小辉数字员工系统总体方案.md)
- [多 Agent 协同方案 V5](docs/xiaohui/MULTI-AGENT-DESIGN-V5.md)
- [商城智能分销商城完整方案](docs/xiaohui/商城智能分销商城完整方案.md)

💬 **对小辉数字员工系统感兴趣的**?欢迎在 [Discussions](https://github.com/zhghub/devloop/discussions) 交流!

---

## 🤝 贡献

DevLoop 是 MIT 协议开源项目。

- 🐛 Bug 反馈: [Issues](https://github.com/zhghub/devloop/issues)
- 💡 功能建议: [Discussions](https://github.com/zhghub/devloop/discussions)
- 🔧 提交代码: 见 [CONTRIBUTING.md](CONTRIBUTING.md)

---

## 📜 License

[MIT](LICENSE) © 2026 DevLoop Contributors

---

🌐 [devloop.cn](https://devloop.cn) · [GitHub](https://github.com/zhghub/devloop) · Discord
