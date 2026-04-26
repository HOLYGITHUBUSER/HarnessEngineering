---
name: hermesagent-deployment
description: Docker deployment baseline for HermesAgent and Hermes Web UI. Use when setting up, restarting, or locating configuration for a local HermesAgent deployment.
---

# HermesAgent 部署技能

## 项目信息

**官方项目**: NousResearch/hermes-agent
- GitHub: https://github.com/NousResearch/hermes-agent
- Stars: 97.4k
- Web UI: EKKOLearnAI/hermes-web-ui

## 部署信息

**部署路径**: `/Users/jl/AAAProgram/Agent-Hermes/hermes-web-ui`

**访问地址**: http://localhost:6060

**访问令牌**: `<your-hermes-webui-token>`

## Docker 部署命令

```bash
cd /Users/jl/AAAProgram/Agent-Hermes/hermes-web-ui
docker compose up -d --build hermes-agent hermes-webui
docker compose logs hermes-webui
```

## 停止服务

```bash
docker compose down
```

## 重启服务

```bash
docker compose restart hermes-webui
```

## 控制台功能

### CLI 命令行界面
- `hermes` - 启动交互式聊天
- `hermes gateway` - 启动消息网关
- `hermes setup` - 配置向导
- `hermes config` - 查看配置

### Web UI 控制台
- AI 聊天界面（实时流式输出）
- 多会话管理
- 平台渠道配置（Telegram、Discord、Slack、WhatsApp、Signal、Email 等）
- 用量分析（Token 统计、费用追踪）
- 定时任务管理
- 模型管理
- 日志查看
- Web 终端

## 配置文件位置

**数据目录**: `hermes_data/`
- 配置: `hermes_data/config.yaml`
- 环境变量: `hermes_data/.env`
- 会话: `hermes_data/sessions/`
- 日志: `hermes_data/logs/`

## 注意事项

- AUTH_DISABLED=true 环境变量已设置，但前端仍要求令牌
- 令牌存储在容器 `/app/packages/server/data/.token`
- 如需重新生成令牌：`docker exec hermes-webui node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`
