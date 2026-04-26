---
name: hermesagent-custom-provider
description: Configure HermesAgent (NousResearch/hermes-agent) to route through an OpenAI-compatible third-party endpoint such as aicodee, 1openclaw, DeepSeek, or any custom gateway. Use when the user reports "API key not loaded", "unknown provider", "401 Missing Authentication header", requests from hermes going to the wrong server (e.g. openrouter.ai instead of the intended endpoint), or when setting up hermes with a non-built-in inference provider.
---

# HermesAgent · Custom Provider 配置权威指南

这份 skill 专门解决一个其他 agent 普遍踩坑的问题：**HermesAgent 要对接一个 OpenAI 兼容的自定义端点（aicodee / 1openclaw / 自建网关等），但不管怎么改 `.env`、`docker-compose` 环境变量都不生效。**

## 关键结论（盲区预警）

**大多数 agent 默认走的错误路径：把 `provider` 写成 `openrouter` + 自定义 `base_url`。这是不通的。**

原因：
- hermes 的 `openrouter` provider 在 `auth.json` 的 credential pool 里会把 `base_url` **硬编码回** `https://openrouter.ai/api/v1`，忽略 `config.yaml` 里的自定义值
- 即便 `OPENROUTER_API_KEY` 环境变量加载成功，请求依然会发到 openrouter 官方而不是你的端点
- `.env` 和 `docker-compose environment:` 字段**不是**问题根因，改它们无济于事

**正确做法：使用 hermes 原生的 `custom_providers` 机制。**

## 快速配置（复制即用）

### Step 1：修改 `hermes_data/config.yaml`（容器内路径 `/home/agent/.hermes/config.yaml`）

把顶部的 `model` 段替换为：

```yaml
custom_providers:
  - name: aicodee              # 自定义的 provider key，后面要引用
    base_url: https://v2.aicodee.com/v1   # 必须带 /v1 后缀（OpenAI 兼容路径）
    api_key: sk-xxxxxxxxxxxx
    models:
      - MiniMax-M2.7-highspeed
      - MiniMax-M2.5-highspeed

model:
  default: MiniMax-M2.7-highspeed
  provider: custom
  custom_provider: aicodee     # 必须与 custom_providers[*].name 一致
  base_url: https://v2.aicodee.com/v1
  api_key: sk-xxxxxxxxxxxx
```

> 建议先备份：`docker exec hermes-agent cp /home/agent/.hermes/config.yaml /home/agent/.hermes/config.yaml.bak-$(date +%s)`

### Step 2：让已运行的 gateway 重载配置

WebUI 容器里运行的 `hermes gateway run --replace` 进程在启动时读一次 config，**改完要重启**才能生效：

```bash
docker restart hermes-webui
```

`hermes-agent` 容器一般在跑 TUI chat，改了 config.yaml 本身不影响 WebUI 链路，但如果你在用 CLI chat，也重启一下：

```bash
docker restart hermes-agent
```

### Step 3：验证

```bash
# 1) hermes 识别到 Custom Endpoint
docker exec hermes-agent /opt/hermes/.venv/bin/hermes status | head -15
# 期望看到： Provider: Custom endpoint   Model: MiniMax-M2.7-highspeed

# 2) gateway health
docker exec hermes-webui node -e 'fetch("http://127.0.0.1:8642/health").then(r=>r.text()).then(console.log)'
# 期望： {"status": "ok", "platform": "hermes-agent"}

# 3) 端到端对话测试（POST /v1/runs，字段叫 input，不是 messages！）
docker exec hermes-webui node -e '
fetch("http://127.0.0.1:8642/v1/runs",{method:"POST",headers:{"Content-Type":"application/json"},
body:JSON.stringify({input:"说你好"})}).then(r=>r.json()).then(console.log)'
# 得到 run_id 后等几秒查事件流：
docker exec hermes-webui node -e '
fetch("http://127.0.0.1:8642/v1/runs/RUN_ID_HERE/events").then(r=>r.text()).then(console.log)'
# 期望看到 message.delta + run.completed，并有模型中文回答
```

## 架构速查（为什么要改这里而不是别处）

```
┌─────────────────────┐         ┌──────────────────────────┐
│  hermes-webui 容器  │         │    hermes-agent 容器      │
│                     │         │                          │
│  Node 服务 :6060 ◄──┼── 浏览器│  （TUI 交互 chat 进程）   │
│        │            │         │                          │
│  spawn hermes gateway run     │                          │
│    监听 127.0.0.1:8642         │                          │
│        │            │         │                          │
│  共享 /home/agent/.hermes（卷 hermes_data）── 同一 config.yaml │
└─────────────────────┘         └──────────────────────────┘
```

关键点：
1. **实际出站请求是 hermes-webui 容器里的 `gateway run` 进程发的**，不是 hermes-agent 容器
2. 两个容器通过 `hermes_data` 卷共享同一份 `config.yaml`，所以改一次就够了
3. `UPSTREAM=http://127.0.0.1:8642` 指的是 **webui 容器自己内部**的 localhost，不是 hermes-agent 容器

## 诊断清单（遇到问题先按顺序查）

| # | 检查项 | 命令 | 判断 |
|---|--------|------|------|
| 1 | 容器都在跑 | `docker ps --filter name=hermes` | 必须看到 `hermes-agent` 和 `hermes-webui` 都 Up |
| 2 | 上游端点本身可用 | 见下方 Python snippet | 直接打 aicodee，排除外部问题 |
| 3 | config.yaml 的 provider | `docker exec hermes-agent head -15 /home/agent/.hermes/config.yaml` | `provider: custom` + `custom_provider: <name>` |
| 4 | hermes 状态识别正确 | `hermes status` | Provider 行应是 `Custom endpoint` |
| 5 | auth.json 未劫持 | `docker exec hermes-agent cat /home/agent/.hermes/auth.json` | `credential_pool.openrouter[].base_url` 不应指向 openrouter.ai（或该池为空） |
| 6 | gateway 进程 | `docker exec hermes-webui ps aux \| grep gateway` | 有 `hermes gateway run --replace` |
| 7 | gateway health | `/health` 应返回 ok | 见上方 Step 3 |

容器里没有 `curl`/`wget`，用 Python 或 Node 代替：

```bash
# 容器外直接打目标端点验证
docker exec hermes-agent /opt/hermes/.venv/bin/python3 -c "
import urllib.request, json
req = urllib.request.Request('https://v2.aicodee.com/v1/models',
    headers={'Authorization':'Bearer sk-XXXX'})
print(urllib.request.urlopen(req, timeout=15).read()[:500].decode())
"
```

## 常见陷阱

### 陷阱 1：把 `provider` 写成 openrouter 配自定义 base_url

```yaml
# ❌ 错误 — 请求会被送到 openrouter.ai
model:
  provider: openrouter
  base_url: https://v2.aicodee.com
```

现象：hermes status 看起来正常，但对话 401 或 "model not found"。原因见开头"关键结论"。

### 陷阱 2：base_url 忘记 `/v1` 后缀

aicodee 这类 OpenAI 兼容端点的完整路径是 `https://host/v1/chat/completions`。base_url 必须是 `https://host/v1`，**不能**只写 `https://host`。

### 陷阱 3：猜测不存在的 provider 名

hermes 的合法 provider（2026-04 版）：
- `custom` — ← 本 skill 推荐
- `openrouter`、`nous`、`openai-codex`、`anthropic`
- **没有** `openai`、`api-key`、`generic`、`http` 这些名字

### 陷阱 4：Run API 的字段名

`POST /v1/runs` 的 body 字段是 `input`（字符串），**不是** `messages`。错了会得到：

```
{"error":{"message":"Missing 'input' field","type":"invalid_request_error"}}
```

### 陷阱 5：改了 .env 期待生效

hermes 的 provider 路由决策在**配置加载期**，`.env` 只对那些"期望从环境变量读密钥"的内置 provider（openrouter/anthropic/…）有效。用 `custom_providers` 把 `api_key` 直接写进 config.yaml 最干净。

### 陷阱 6：WebUI 看起来没变化

改完 config.yaml 一定要 `docker restart hermes-webui`，因为 gateway 进程在启动时读一次配置并缓存。

## 参考片段：已验证工作的完整 config.yaml 顶部

```yaml
custom_providers:
  - name: aicodee
    base_url: https://v2.aicodee.com/v1
    api_key: sk-REPLACE_WITH_YOUR_AICODEE_KEY
    models:
      - MiniMax-M2.7-highspeed
      - MiniMax-M2.5-highspeed

model:
  default: MiniMax-M2.7-highspeed
  provider: custom
  custom_provider: aicodee
  base_url: https://v2.aicodee.com/v1
  api_key: sk-REPLACE_WITH_YOUR_AICODEE_KEY

platforms:
  api_server:
    enabled: true
    key: ''
    cors_origins: '*'
    extra:
      port: 8642
      host: 127.0.0.1
```

## 适配其他端点

| 端点 | base_url | 常见模型 |
|------|----------|---------|
| aicodee v2 | `https://v2.aicodee.com/v1` | `MiniMax-M2.7-highspeed` |
| 1openclaw | `https://www.1openclaw.cn/v1` | 取决于账户 |
| DeepSeek 官方 | `https://api.deepseek.com/v1` | `deepseek-chat` |
| 任意自建 OpenAI 兼容网关 | `https://your-host/v1` | 视实现而定 |

把上面 `custom_providers` 里的 `name / base_url / api_key / models` 改成对应值即可，其他结构不动。

## 相关文件

- 部署基线：`../hermesagent-deployment-部署技能.md`
- 调试命令参考：`../hermesagent-对话调试-troubleshooting.md`（注意：该文件末尾的"未解决"状态已过时，真正解法以本 skill 为准）
- aicodee 账户信息：`../aicodee-api-配置信息.md`
