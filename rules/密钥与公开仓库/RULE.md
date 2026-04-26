# 密钥与公开仓库

- 绝不能提交真实 API key、密码、token、私有 cookie 或凭据文件。
- 公开 GitHub 仓库默认视为不安全环境，不能放任何真实密钥。
- 添加 skills、文档、日志、截图、配置示例前，先扫描是否包含凭据。
- 敏感值要替换成占位符，例如 `sk-REPLACE_WITH_YOUR_KEY`、`<your-password>`、`<your-token>`。
- 如果密钥已经暴露，要提醒用户立即轮换密钥。
- 文件一旦被 git 跟踪，不能只靠 `.gitignore` 解决，必须从索引或历史里处理。
