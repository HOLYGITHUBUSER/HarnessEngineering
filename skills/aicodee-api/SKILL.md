---
name: aicodee-api-template
description: Sanitized aicodee API endpoint configuration template. Use when documenting or configuring an OpenAI-compatible aicodee endpoint without committing real keys, accounts, or passwords.
---

# aicodee API 配置信息

## API 基本信息

**API Base URL**: `https://v2.aicodee.com`

**API Key**: `sk-REPLACE_WITH_YOUR_AICODEE_KEY`

**文档地址**: https://lcnwoe31c51t.feishu.cn/wiki/A892wYxnVippRXkviJRcw7LBnGV?from=from_copylink

## 账户信息

**账户**: `<your-account>`

**密码**: `<your-password>`

**登录后可以查看使用量，登录后请修改密码**

## 支持的模型

- `MiniMax-M2.5-highspeed`
- `MiniMax-M2.7-highspeed`

## 用量查询

**用量查询地址**: http://v2api.aicodee.com/chaxun

## 配置到 HermesAgent

在 `hermes_data/.env` 文件中添加：

```bash
# aicodee API 配置
OPENROUTER_API_KEY=sk-REPLACE_WITH_YOUR_AICODEE_KEY
OPENROUTER_BASE_URL=https://v2.aicodee.com
```

或修改 `hermes_data/config.yaml` 中的模型配置：

```yaml
model:
  default: MiniMax-M2.7-highspeed
  provider: openrouter
  base_url: https://v2.aicodee.com
```

## 注意事项

- API Key 首用激活，激活后 24 小时失效
- 登录后请及时修改密码
- 定期查看用量，避免超出配额
