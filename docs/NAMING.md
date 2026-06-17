# 🏷️ 命名说明 (NAMING)

> DevLoop vs DevLog — 别搞混了!

---

## 一句话

> **DevLog = DevLoop 的旧名**(2026-06-17 正式改名)

---

## 📜 命名时间线

| 日期 | 事件 | 名字 |
|---|---|---|
| 2026-04-25 | 小辉数字员工 2.0 内部模块诞生 | `DevLog` |
| 2026-05-12 | V2.0 智能分发 | `DevLog` |
| 2026-06-02 | V3.0 AI 原生架构 | `DevLog` |
| 2026-06-09 | V2.3 SMART-DISPATCH 主调度 | `DevLog` |
| 2026-06-12 | V3.1 终态 (8 角色/代际审查) | `DevLog` |
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

## 🔄 改名影响

### ✅ 已改
- GitHub 仓库: `zhghub/devloop`
- 域名: `devloop.cn`
- 描述 / Topics / Logo
- 文档文件名: `DEVLOG-*.md` → `DEVLOOP-*.md`
- API 双轨: `/api/v1/dev` (旧) + `/api/v1/devloop` (新)

### ⏳ 待改 (Stage 2 代码剥离时)
- 数据库表前缀: `dev_*` → `devloop_*` (或保留)
- 模块目录: `modules/devlog/` → `modules/devloop/`
- npm 包名
- Docker 镜像名
- 部分文档正文(讲 "DevLog" 历史的,不改;讲 "DevLoop" 现在的,改)

---

## 📚 文档里为什么还有 "DevLog" 字眼?

**故意的**,原因:

1. **历史连贯性** — 文档讲的是 V3.1 时代架构,那时还叫 DevLog
2. **架构代码引用** — 文档里大量引用 `dev_*` 表 / `devlog/` 目录,改了会破坏文档可读性
3. **过渡期** — 等 Stage 2 代码剥离完,所有文档会二次重写,统一用 DevLoop

**简单说**:
> 文档里说的 `DevLog` = 现在的 `DevLoop`,是同一个东西的不同时期名字。
> 看到 `DevLog` 不要以为是另一个项目,就是 DevLoop 旧称。

---

## 🧪 怎么验证是同一个项目?

任何文档 / 代码看到 `DevLog`,做这两个检查之一就能确认是 DevLoop:

1. **看仓库名** — 如果在 `zhghub/devloop` 仓库里,就是同一个
2. **看 API 路径** — `/api/v1/dev/*` 或 `/api/v1/devloop/*` 双轨都是

---

## 📅 改名完成度

| 维度 | 进度 | 备注 |
|---|---|---|
| GitHub 仓库 | ✅ 100% | devloop |
| 域名 | ✅ 100% | devloop.cn |
| 文档**文件名** | ✅ 100% | DEVLOOP-* |
| 文档**正文** | ⏳ 30% | 仅过渡说明 + README 引用 |
| 数据库表 | ⏳ 0% | Stage 2 剥离时改 |
| 代码模块 | ⏳ 0% | Stage 2 剥离时改 |

---

## ❓ FAQ

### Q: 改名前的 commit history 会保留吗?

**A**: ✅ 保留。`git log` 能看到完整历史,只是仓库名变了。

### Q: 看到旧 commit 里说 "DevLog" 是 bug 吗?

**A**: ❌ 不是 bug。那是历史 commit,当时还叫 DevLog。

### Q: 什么时候正文也会改?

**A**: Stage 2 (代码剥离, ~2026-07)。届时所有文档会基于实际代码二次重写。

---

🔄 [devloop.cn](https://devloop.cn) · [GitHub](https://github.com/zhghub/devloop)
