# 🗺️ DevLoop Roadmap

> 公开路线图。承诺不一定都按时,但方向不变。
> 最后更新: 2026-06-17

## 🎯 愿景

让 AI Agent 像一支团队协作你的 Git 项目。

## 📍 当前状态 (2026-06)

**v0.1.0 仓库预热** ✅

- ✅ GitHub 仓库建立
- ✅ README + 完整方案文档公开
- ✅ CI workflow 就绪
- ⏳ 代码还没从小辉数字员工系统剥离

## 🚦 三阶段路线

### Stage 1: 仓库预热 (2026-06, **当前**)

**目标**: 让社区看到 DevLoop 想做什么、怎么做

| ✅ | 任务 | 状态 |
|---|---|---|
| ✅ | GitHub 仓库 + README + LICENSE | 完成 |
| ✅ | 11 个仓库元数据 (Description/Topics/主页) | 完成 |
| ✅ | CI workflow (Node 22 + PG 15 + Redis 7) | 完成 |
| ✅ | 8 份方案文档公开 (`docs/`) | 完成 |
| ✅ | ROADMAP + ARCHITECTURE + DESIGN_DECISIONS | 完成 |

### Stage 2: 代码剥离 (2026-07 ~ 2026-08, 4-6 周)

**目标**: 从小辉数字员工 2.0 系统剥离 devlog 模块,做出 standalone 版 DevLoop

| 工作量 | 任务 |
|---|---|
| 1.1 (2h) | 新建独立仓库,拷出 devlog 模块 (109 文件) |
| 1.2 (1h) | 写独立 `package.json` (~12-15 依赖) |
| 1.3 (2h) | 拷/抽象 5 个通用 lib (db/logger/heartbeat/...) |
| 1.4 (1h) | 自实现 `requireInternal` middleware |
| 1.5 (0.5h) | 替换 money-constitution 引用 |
| 1.6 (2h) | feishu-push-service 引用替换 |
| 1.7 (2h) | 写 standalone install.sh + docker-compose.yml |
| 2.1 (2h) | 一键部署测通 |
| 2.2 (2h) | 写 README 30 秒上手 + docker run 一行命令 |
| 2.3 (3h) | 写完整 API 文档 + 文档站 |
| 3.1 (1h) | 跑通 P0 smoke test |
| 3.2 (1h) | Gitee 公开仓库 + 写公告 |

**小计: 约 20 小时, 2-3 个工作日**

### Stage 3: V3.1 P0 补完 (2026-09 ~ 2026-10, 1 个月)

**目标**: 4 入口接入 + 代际进化审查 + 自修复闭环

| 优先级 | 工作量 | 任务 |
|---|---|---|
| P0 | 7h | 4 入口接入 (intake-agent 缺失 / 4 通道识别 / 4 通道桥) |
| P0 | 4h | 代际 prompt 改造 (review.js 带上次结果) |
| P1 | 35h | INTAKE 4 + GEN 5 + 表盘自演化 3 + 弹药库 5 |
| P2 | 30h | 6 角色独立 agent + 代际训练数据 |

### Stage 4: 1.0 GA + 公开 SaaS (2026-11, 1 个月)

- ✅ V3.1 P0 + P1 完整
- ✅ 公开 SaaS 上线
- ✅ 多租户支持
- ✅ 计费模型
- ✅ 异地 Agent SDK

## 🎯 关键里程碑

| 日期 | 里程碑 |
|---|---|
| 2026-06-17 | v0.1.0 仓库预热 ✅ |
| 2026-07 | v0.2.0 代码剥离完成 |
| 2026-08 | v0.3.0 V3.1 P0 完整 |
| 2026-09 | v0.4.0 V3.1 P1 完整 |
| 2026-10 | v0.5.0 V3.1 P2 完整 |
| 2026-11 | **v1.0.0 GA + 公开 SaaS** |

## 🤝 贡献者指南

想参与?看 [CONTRIBUTING.md](../CONTRIBUTING.md)

特别欢迎:
- 🐛 Bug 反馈
- 📝 文档改进
- 🌍 多语言翻译 (英文 README)
- 🧪 测试用例
- 💡 功能建议

## 📬 联系我们

- 💬 [GitHub Discussions](https://github.com/zhghub/devloop/discussions)
- 🐛 [GitHub Issues](https://github.com/zhghub/devloop/issues)
- 🌐 [devloop.cn](https://devloop.cn)

---

🔄 [devloop.cn](https://devloop.cn) · [GitHub](https://github.com/zhghub/devloop)
