# ❓ DevLoop FAQ

> 常见问题快速答疑。问题没在下面?去 [GitHub Discussions](https://github.com/zhghub/devloop/discussions) 提问。

---

## 🤔 关于 DevLoop 是什么

### Q1: DevLoop 是什么?

**A**: DevLoop = **让 AI Agent 像一支团队协作你的 Git 项目** 的开源调度中枢。
8 角色 (leader/developer/reviewer/auditor/...) + 196 REST API + 12 ACP harness 兼容。

### Q2: 跟 LangChain / AutoGen / CrewAI 有什么区别?

**A**: 那些是 **agent 框架** (SDK 嵌入式)。
DevLoop 是 **项目协作中枢** (REST API 全开放,数据可查可控)。

| 维度 | LangChain/CrewAI | DevLoop |
|---|---|---|
| 数据可查 | 黑盒 | 196 REST API 全开放 |
| 多角色协作 | 临时编排 | 8 角色固定 |
| 审查学习 | 一次性 LLM | 代际进化 (越用越准) |
| LLM 锁定 | 多家兼容 | 12 个 ACP harness |
| 部署方式 | SDK 嵌入 | Docker compose 一键 |

### Q3: 跟 Aider / Devin 有什么区别?

**A**: 那些是 **单 agent 工具**。
DevLoop 是 **多 agent 团队调度**,内置 leader/developer/reviewer/auditor 分工。

### Q4: DevLoop 跟小辉数字员工是什么关系?

**A**: DevLoop 是从 **小辉数字员工 2.0** 系统里剥离出来的开源模块。
- ✅ DevLoop 拿走了: 数据底座 + 智能体调度 (0 业务耦合)
- ❌ DevLoop 不包含: 小辉的 CRM/ERP/商城等业务模块

详细看 [docs/xiaohui/README.md](xiaohui/README.md)

---

## 🛠️ 关于使用

### Q5: 怎么安装?

**A**: 等代码剥离完 (Stage 2,~ 2026-07)。
现在 (v0.1.0 仓库预热) 只有文档。

预计安装流程:
```bash
git clone https://github.com/zhghub/devloop
cd devloop
docker compose up -d
curl http://localhost:3001/api/v1/dev/health
```

### Q6: 需要哪些依赖?

**A**:
- Docker + Docker Compose
- PostgreSQL 15+ (Docker 自动起)
- Redis 7+ (Docker 自动起)
- Node.js 22+ (本地开发时需要)

### Q7: 支持哪些 LLM?

**A**: 12 个 [ACP harness](ACP-AGENT-CAPABILITY-MATRIX.md):
- 第一档 (内置): openclaw
- 第二档 (npm install): claude / codex / opencode / gemini
- 第三档 (后续): copilot / cursor / droid / iflow / kilocode / kimi / kiro / qwen

切换 LLM 不用改代码,改 `~/.acpx/config.json` 就行。

### Q8: 有 SaaS 版吗?

**A**: 计划中。Roadmap 见 [ROADMAP.md](ROADMAP.md)。
- v0.1.0 (现在): 文档预热
- v1.0 GA (~2026-11): 开源 + 自部署
- SaaS 上线: 1.0 之后

---

## 🤝 关于参与

### Q9: 怎么贡献代码?

**A**: 看 [CONTRIBUTING.md](../CONTRIBUTING.md)
1. Fork 仓库
2. 创建分支 (`git checkout -b feature/xxx`)
3. 写代码 + 测试
4. 提 PR

### Q10: 我不会写代码,能帮什么忙?

**A**: 很多!
- 🐛 报告 Bug
- 📝 改进文档 (错别字/翻译/示例)
- 💡 提功能建议
- 🧪 写测试用例
- 🌍 多语言翻译
- 📢 帮宣传 (推文/博客)

### Q11: 商用可以吗?

**A**: ✅ **可以**,MIT 协议是最宽松的,允许商用、修改、私有化。
唯一要求:保留版权声明。

---

## 📞 联系方式 / 客服渠道

### 现在 (v0.1.0)

| 渠道 | 用途 | 链接 |
|---|---|---|
| 💬 GitHub Discussions | 技术问题 / 想法 / 分享 | https://github.com/zhghub/devloop/discussions |
| 🐛 GitHub Issues | Bug 报告 / 功能建议 | https://github.com/zhghub/devloop/issues |
| 🌐 官网 | 项目主页 | https://devloop.cn |

### 即将上线

| 渠道 | 状态 | 预计时间 |
|---|---|---|
| 📱 小辉数字员工 Webchat 客服 | 🔜 待上线 | v1.0 GA 后 (~2026-11) |
| 🎵 抖音号 | 🔜 待开通 | 同步进行中 |
| 💬 Discord | 🔜 待开通 | v1.0 GA 时 |

> 💡 **webchat 上线后**,这里会贴直达链接,GitHub Issues/Discussions 也会自动引导到小辉客服。

---

## 🆘 还有问题?

- 💬 [开 Discussion](https://github.com/zhghub/devloop/discussions/new?category=q-a) (推荐)
- 🐛 [提 Issue](https://github.com/zhghub/devloop/issues/new)
- 📖 看 [ROADMAP.md](ROADMAP.md) / [ARCHITECTURE.md](ARCHITECTURE.md)

---

🔄 [devloop.cn](https://devloop.cn) · [GitHub](https://github.com/zhghub/devloop)
