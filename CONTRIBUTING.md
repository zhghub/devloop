# 🤝 贡献指南

欢迎来到 DevLoop!我们很高兴你想贡献 🎉

## 📜 行为准则

本项目遵守 [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md)。
参与即表示你同意遵守。

## 🐛 报告 Bug

用 [Bug Report 模板](.github/ISSUE_TEMPLATE/bug_report.md) 提交 Issue。

## 💡 功能建议

用 [Feature Request 模板](.github/ISSUE_TEMPLATE/feature_request.md) 提交 Issue。

## 🔧 提交代码

1. **Fork 仓库**
2. **创建分支** (`git checkout -b feature/xxx`)
3. **写代码 + 写测试**
4. **跑测试** (`npm test`)
5. **提交** (`git commit -m "feat: xxx"`)
6. **推送** (`git push origin feature/xxx`)
7. **提 PR** (用 [PR 模板](.github/PULL_REQUEST_TEMPLATE.md))

## 📝 Commit 规范

遵循 [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` 新功能
- `fix:` Bug 修复
- `docs:` 文档变更
- `refactor:` 重构
- `test:` 测试
- `chore:` 杂项

## 🧪 本地开发

```bash
# 1. Clone
git clone https://github.com/devloop/devloop
cd devloop

# 2. 安装依赖
npm install

# 3. 起 PostgreSQL + Redis
docker compose up -d postgres redis

# 4. 跑 migration
npm run migrate

# 5. 启动
npm run dev
```

## 📬 联系我们

- 💬 [GitHub Discussions](https://github.com/devloop/devloop/discussions)
- 🐛 [GitHub Issues](https://github.com/devloop/devloop/issues)

---

感谢你的贡献!❤️
