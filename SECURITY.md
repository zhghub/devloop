# 🔒 Security Policy

## 支持的版本

| 版本 | 支持状态 |
|---|---|
| 0.1.x (开发中) | ✅ 活跃开发 |
| < 0.1 | ❌ 不支持 |

## 报告漏洞

我们非常重视安全问题。请**不要**在公开 Issue 中报告安全漏洞。

📧 邮箱:kmjdk@qq.com

### 报告内容

请包含:

1. **漏洞描述** — 简短说明
2. **复现步骤** — 详细步骤
3. **影响范围** — 受影响的版本
4. **PoC** — 概念验证代码 (如果有)
5. **修复建议** (可选)

### 响应时间

- ✅ 24 小时内确认收到
- 🔍 7 天内评估严重性
- 🛠️ 90 天内修复高危漏洞

## 安全最佳实践

部署 DevLoop 时:

1. **修改默认密码** — `INTERNAL_API_SECRET` 不要用默认值
2. **限制网络访问** — 仅信任来源 IP
3. **启用 HTTPS** — 生产环境必须
4. **定期更新** — 订阅 [Releases](https://github.com/devloop/devloop/releases)
5. **审计日志** — 定期检查 `/api/v1/dev/audit-runs`

## 致谢

负责任披露漏洞的贡献者会在 [SECURITY_ACKNOWLEDGMENTS.md](SECURITY_ACKNOWLEDGMENTS.md) 中致谢。

---

🔄 [devloop.cn](https://devloop.cn) · [GitHub](https://github.com/devloop/devloop)
