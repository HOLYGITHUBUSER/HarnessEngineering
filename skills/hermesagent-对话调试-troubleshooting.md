# HermesAgent 对话调试技能

## 项目信息

**官方项目**: NousResearch/hermes-agent
- GitHub: https://github.com/NousResearch/hermes-agent
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

## 对话调试步骤

### 1. 检查容器运行状态

```bash
docker ps
docker compose logs hermes-agent --tail 30
```

### 2. 检查配置文件

#### 查看 config.yaml

```bash
docker exec hermes-agent sh -c "cat /home/agent/.hermes/config.yaml"
```

**关键配置项**：
```yaml
model:
  default: MiniMax-M2.7-highspeed
  provider: openrouter  # 支持: openrouter, nous, local/custom, openai-codex
  base_url: https://v2.aicodee.com
  api_key: sk-REPLACE_WITH_YOUR_AICODEE_KEY  # 可选：直接在配置中设置
```

#### 查看 .env 文件

```bash
docker exec hermes-agent sh -c "cat /home/agent/.hermes/.env"
```

**环境变量配置**：
```bash
OPENROUTER_API_KEY=sk-REPLACE_WITH_YOUR_AICODEE_KEY
OPENROUTER_BASE_URL=https://v2.aicodee.com
```

### 3. 检查错误日志

```bash
docker exec hermes-agent sh -c "tail -50 /home/agent/.hermes/logs/errors.log"
docker exec hermes-agent sh -c "tail -50 /home/agent/.hermes/logs/agent.log"
```

**常见错误**：
- `OPENROUTER_API_KEY not set` - 环境变量未正确加载
- `unknown provider 'openai'` - provider 名称不支持
- `Error code: 401 - Missing Authentication header` - API Key 无效或未传递

### 4. 直接测试 API

测试 aicodee API：
```bash
curl -X POST https://v2.aicodee.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-REPLACE_WITH_YOUR_AICODEE_KEY" \
  -d '{
    "model": "MiniMax-M2.7-highspeed",
    "messages": [{"role": "user", "content": "你好"}],
    "max_tokens": 50
  }'
```

### 5. 容器内测试对话

```bash
docker exec hermes-agent sh -c "echo '你好' | /opt/hermes/.venv/bin/hermes"
```

### 6. 修改配置方法

#### 方法 1: 修改 config.yaml

```bash
docker exec hermes-agent sh -c "sed -i 's/provider: openrouter/provider: api-key/g' /home/agent/.hermes/config.yaml"
docker exec hermes-agent sh -c "sed -i 's|base_url: https://v2.aicodee.com|base_url: https://www.1openclaw.cn/v1|g' /home/agent/.hermes/config.yaml"
```

#### 方法 2: 修改 .env 文件

```bash
docker exec hermes-agent sh -c "cat > /home/agent/.hermes/.env << 'EOF'
OPENROUTER_API_KEY=sk-REPLACE_WITH_YOUR_AICODEE_KEY
OPENROUTER_BASE_URL=https://v2.aicodee.com
EOF"
```

#### 方法 3: 修改 docker-compose.yml

```yaml
services:
  hermes-agent:
    environment:
      - OPENROUTER_API_KEY=sk-REPLACE_WITH_YOUR_AICODEE_KEY
      - OPENROUTER_BASE_URL=https://v2.aicodee.com
```

### 7. 重启服务

```bash
docker restart hermes-agent
# 或
docker compose restart hermes-agent
```

### 8. 交互式配置

进入容器交互式配置：
```bash
docker exec -it hermes-agent /opt/hermes/.venv/bin/hermes model
```

## API 服务配置

### aicodee API

**API Base URL**: `https://v2.aicodee.com`
**API Key**: `sk-REPLACE_WITH_YOUR_AICODEE_KEY`
**模型**: MiniMax-M2.7-highspeed

### 1openclaw API

**API Base URL**: `https://www.1openclaw.cn/v1`
**API Key**: `sk-REPLACE_WITH_YOUR_1OPENCLAW_KEY`
**状态**: 令牌已过期

## 已尝试的配置方法

1. ✅ docker-compose.yml 环境变量 - OPENROUTER_API_KEY
2. ✅ .env 文件配置 - OPENROUTER_API_KEY
3. ✅ config.yaml 直接配置 api_key
4. ❌ provider: openai - 不支持
5. ❌ provider: api-key - 不支持
6. ✅ provider: openrouter - 支持但环境变量加载有问题

## 当前状态（2026-04-18 更新）

- ✅ HermesAgent 部署成功
- ✅ Web UI 运行正常（v0.3.5）
- ✅ aicodee API 对接成功（通过 `custom_providers` 方案）
- ✅ 端到端对话链路已打通（`POST /v1/runs` 正常返回中文回答）

## 正确解法（权威）

见 `hermesagent-custom-provider配置/SKILL.md`。

**核心要点**（避免再走弯路）：
- ❌ 不要把 `provider` 写成 `openrouter` 配自定义 `base_url`：`auth.json` 里 openrouter 凭据池会硬编码回 openrouter.ai，请求发不到目标端点
- ❌ 不要依赖 `.env` / docker-compose `environment:` 来注入 API key：对自定义端点无效
- ✅ 使用 `custom_providers` + `model.provider: custom` + `model.custom_provider: <name>`
- ✅ 改完 `config.yaml` 必须 `docker restart hermes-webui`（gateway 进程启动时缓存配置）

## 注意事项

- HermesAgent 合法 provider：`custom`、`openrouter`、`nous`、`openai-codex`、`anthropic`
- **没有** `openai`、`api-key`、`generic`、`http` 这些名字
- 对接任意 OpenAI 兼容第三方端点（aicodee、1openclaw、DeepSeek、自建网关），一律走 `custom` provider
- base_url 要带完整的 `/v1` 后缀
- `POST /v1/runs` 的 body 字段是 `input`（字符串），不是 `messages`
