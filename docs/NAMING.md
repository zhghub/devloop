# 🏷️ 命名历史 (从 DevLog 到 DevLoop)

> DevLoop (新名) vs DevLog (旧名) — 历史档案,记录改名全过程。

---

## ✅ 一句话总结

> **DevLog** (旧名,2026-06-17 改名) → **DevLoop** (新名)
>
> 改名已**100% 完成**(文档层 + 仓库层),仅剩代码层 (Stage 2 剥离时改)。

---

## 📜 命名时间线

| 日期 | 事件 | 名字 |
|---|---|---|
| 2026-04-25 | 小辉数字员工 2.0 内部模块诞生 | `DevLog` |
| 2026-05-12 | V2.0 智能分发 | `DevLog` |
| 2026-06-02 | V3.0 AI 原生架构 | `DevLog` |
| 2026-06-09 | V2.3 SMART-DISPATCH 主调度 | `DevLog` |
| 2026-06-12 | V3.1 终态 (8 角色 / 代际审查) | `DevLog` |
| **2026-06-17** | **正式独立开源 + 改名** | **`DevLoop`** ✨ |

---

## 🤔 为什么改?

| 旧名 `DevLog` | 新名 `DevLoop` |
|---|---|
| "开发日志" — 像工具名 | "开发循环" — 像系统/生态 |
| 嵌入小辉内部 | 独立开源产品 |
| 中文社区为主 | 国内 + 海外通用 |
| 含义偏工具 | 含义偏平台/调度中枢 |

---

## 🔄 改名完成情况

### ✅ 已 100% 改完 (文档层)
- GitHub 仓库名: `zhghub/devloop` ✅
- 域名: `devloop.cn` ✅
- 仓库元数据: Description / Topics / Logo ✅
- 文档**文件名**: `DEVLOG-*.md` → `DEVLOOP-*.md` ✅ (git mv 保留历史)
- 文档**正文**: DevLog → DevLoop ✅ (本批次完成)
- README / FAQ / NAMING / 欢迎帖 ✅

### ⏳ 待改 (代码层, Stage 2 剥离时)
- 数据库表前缀: `dev_*` (保留还是 `devloop_*` 待定)
- 模块目录: `modules/devlog/` → `modules/devloop/`
- npm 包名 / Docker 镜像名
- 类名 / 变量名里的 "DevLog"

---

## 📚 文档里为什么还有 "dev_*" / "devlog/" 字眼?

**故意的**,原因:

1. **代码引用** — 文档里大量引用 `dev_task_logs` / `dev_spawn_queue` / `dev_agent_registry` 等 16 张表
2. **路径引用** — `modules/devlog/` 目录、`acpx/devloop` 路径等
3. **准确性** — 改了就跟实际代码对不上了

**简单说**:
> 文档正文现在 100% 用 **DevLoop** (项目名)
> 但代码引用保留 `dev_*` 前缀 (表名/字段名)
> 看到 `dev_*` 不是旧名,是**代码层的命名约定**

---

## 🧪 怎么确认在 DevLoop 仓库?

任何文档 / 代码出现"DevLoop"或"DevLog",做这两个检查之一就能确认是 DevLoop 项目:

1. **看仓库名** — 如果在 `zhghub/devloop` 仓库里,就是
2. **看 API 路径** — `/api/v1/dev/*` 或 `/api/v1/devloop/*` 双轨都是

---

## 📅 改名里程碑

| 时刻 | 事件 |
|---|---|
| 2026-06-17 上午 | 命名拍板 (`hgdevhub` 占位 → `DevLoop`) |
| 2026-06-17 中午 | 仓库创建 + README + LICENSE |
| 2026-06-17 下午 | 文档预热 + Discussions + FAQ |
| 2026-06-17 下午 | 命名混淆解决 (DEVLOG-* 改名 + 加 NAMING.md) |
| 2026-06-17 下午 | **本批次: 文档正文 DevLog → DevLoop 全量替换** ✨ |
| ~2026-07 (Stage 2) | 代码层改名 (表 / 目录 / 类名) |

---

## ❓ FAQ

### Q: 改名前的 commit history 会保留吗?

**A**: ✅ 保留。`git log` 能看到完整历史,只是仓库名 + 文档变了。

### Q: 看到旧 commit 里说 "DevLog" 是 bug 吗?

**A**: ❌ 不是 bug。那是 2026-06-17 改名前的历史 commit,当时确实叫 DevLog。

### Q: 文档正文里还有 "DevLog" 字眼吗?

**A**: 改名前有 ~80 处,**本批次已全量替换为 DevLoop**。如发现漏网之鱼,欢迎提 Issue。

### Q: 代码层什么时候改?

**A**: Stage 2 (代码剥离, ~2026-07)。届时基于实际代码二次重写,所有 `dev_*` 表 / `devlog/` 路径会决策保留或改名。

---

🔄 [devloop.cn](https://devloop.cn) · [GitHub](https://github.com/zhghub/devloop)
