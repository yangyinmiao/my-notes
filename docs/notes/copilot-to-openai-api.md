# GitHub Copilot 转 OpenAI API 部署指南

## 概述

本文档介绍如何将 GitHub Copilot API 转换为 OpenAI 兼容的 API 接口，通过容器化部署实现协议转换和安全访问控制。

## 系统架构

```
客户端请求 (OpenAI 格式)
    ↓
鉴权代理容器 (端口 12788, HTTPS/HTTP)
    ↓ [验证 API Key]
copilot-api 容器 (端口 9723, 本地)
    ↓ [协议转换]
GitHub Copilot API
    ↓
返回响应
```

## 核心原理

### 1. copilot-api-pro 的作用

这是核心转换服务，负责：

- **协议适配**：将 OpenAI 标准格式 (`/v1/chat/completions`) 转换为 GitHub Copilot 的内部 API 格式
- **会话管理**：维护与 GitHub Copilot 服务的认证会话
- **模型映射**：将请求的模型名映射到 Copilot 实际支持的模型
- **响应转换**：将 Copilot 的响应转换回 OpenAI 格式

### 2. 鉴权代理的作用

提供安全层：

- copilot-api 本身没有鉴权机制，只监听本地 9723 端口
- 代理层通过 `Bearer Token` 验证调用者身份
- 可以添加 HTTPS 支持，保护传输安全

## 部署步骤

### 步骤 1：部署 copilot-api 容器

```bash
# 安装 podman 和依赖
apt install -y podman fuse-overlayfs

# 删除旧容器（如果存在）
podman rm -f copilot-proxy

# 启动 copilot-api-pro 容器（支持 v1/response 接口）
podman run -d \
  --restart=always \
  --name copilot-proxy \
  -p 9723:9723 \
  docker.io/node:20-slim \
  sh -c "npm install -g copilot-api-pro && copilot-api-pro start --port 9723"
```

**说明**：
- 使用 `copilot-api-pro` 而非普通版本，支持更多接口
- 端口 9723 仅本地监听，不对外暴露
- 使用 `--restart=always` 确保容器自动重启

### 步骤 2：GitHub 账号授权

#### 2.1 查看设备码

```bash
podman logs -f copilot-proxy
```

输出示例：

```
ℹ Using VSCode version: 1.115.0
ℹ Not logged in, getting new access token
ℹ Please enter the code "XXXX-XXXX" in https://github.com/login/device
```

#### 2.2 授权设备

1. 访问 [https://github.com/login/device](https://github.com/login/device)
2. 输入设备码 `XXXX-XXXX`
3. 授权你的 GitHub 账户

#### 2.3 验证授权成功

授权成功后，日志会显示：

```
ℹ Logged in as YOUR_USERNAME
ℹ Available models:
- claude-opus-4.6
- claude-sonnet-4.6
- gpt-4o
- gpt-4o-mini
- gpt-5.2
- gemini-3-flash-preview
...

➜ Listening on: http://localhost:9723/ (all interfaces)
```

### 步骤 3：部署鉴权代理容器

根据部署环境选择：

#### 选项 A：HTTPS 鉴权容器（用于公网服务器）

```bash
podman run -d \
  --restart=always \
  --name copilot-auth-proxy \
  -e API_KEY="<YOUR_API_KEY>" \
  -e UPSTREAM_URL="http://127.0.0.1:9723" \
  -v <YOUR_CERT_DIR>:/certs:ro \
  --net=host \
  docker.io/python:3.12-slim \
  sh -c "pip install aiohttp -q && python -c '
import os, ssl, aiohttp
from aiohttp import web

API_KEY      = os.environ[\"API_KEY\"]
UPSTREAM_URL = os.environ.get(\"UPSTREAM_URL\", \"http://127.0.0.1:9723\")
PORT         = int(os.environ.get(\"PORT\", 12788))
SKIP_HEADERS = {\"host\", \"authorization\", \"transfer-encoding\", \"content-length\"}

ssl_ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
ssl_ctx.load_cert_chain(\"/certs/fullchain.cer\", \"/certs/private.key\")
ssl_ctx.minimum_version = ssl.TLSVersion.TLSv1_2

async def proxy(request):
    auth = request.headers.get(\"Authorization\", \"\")
    if not (auth.startswith(\"Bearer \") and auth.split(\" \", 1)[1] == API_KEY):
        return web.json_response({\"error\": \"Unauthorized\"}, status=401)
    target_url = UPSTREAM_URL + request.path_qs
    fwd_headers = {k: v for k, v in request.headers.items() if k.lower() not in SKIP_HEADERS}
    body = await request.read()
    async with aiohttp.ClientSession() as session:
        async with session.request(request.method, target_url, headers=fwd_headers, data=body, allow_redirects=False) as up:
            resp_headers = {k: v for k, v in up.headers.items() if k.lower() not in {\"transfer-encoding\", \"content-length\"}}
            response = web.StreamResponse(status=up.status, headers=resp_headers)
            await response.prepare(request)
            async for chunk in up.content.iter_any():
                await response.write(chunk)
            await response.write_eof()
            return response

app = web.Application()
app.router.add_route(\"*\", \"/{path:.*}\", proxy)
print(f\"HTTPS proxy listening on 0.0.0.0:{PORT} -> {UPSTREAM_URL}\")
web.run_app(app, host=\"0.0.0.0\", port=PORT, ssl_context=ssl_ctx)
'"
```

**环境变量说明**：

- `API_KEY`：设置一个强密码作为 API 密钥
- `UPSTREAM_URL`：copilot-api 的地址，默认 `http://127.0.0.1:9723`
- `PORT`：代理监听端口，默认 12788
- `<YOUR_CERT_DIR>`：SSL 证书目录，需包含 `fullchain.cer` 和 `private.key`

#### 选项 B：HTTP 鉴权容器（用于内网服务器）

```bash
podman run -d \
  --restart=always \
  --name copilot-auth-proxy \
  -e API_KEY="<SET_A_API_KEY>" \
  -e UPSTREAM_URL="http://127.0.0.1:9723" \
  --net=host \
  docker.io/python:3.12-slim \
  sh -c "pip install aiohttp -q && python -c '
import os, aiohttp
from aiohttp import web

API_KEY      = os.environ[\"API_KEY\"]
UPSTREAM_URL = os.environ.get(\"UPSTREAM_URL\", \"http://127.0.0.1:9723\")
PORT         = int(os.environ.get(\"PORT\", 12788))
SKIP_HEADERS = {\"host\", \"authorization\", \"transfer-encoding\", \"content-length\"}

async def proxy(request):
    auth = request.headers.get(\"Authorization\", \"\")
    if not (auth.startswith(\"Bearer \") and auth.split(\" \", 1)[1] == API_KEY):
        return web.json_response({\"error\": \"Unauthorized\"}, status=401)
    target_url = UPSTREAM_URL + request.path_qs
    fwd_headers = {k: v for k, v in request.headers.items() if k.lower() not in SKIP_HEADERS}
    body = await request.read()
    async with aiohttp.ClientSession() as session:
        async with session.request(request.method, target_url, headers=fwd_headers, data=body, allow_redirects=False) as up:
            resp_headers = {k: v for k, v in up.headers.items() if k.lower() not in {\"transfer-encoding\", \"content-length\"}}
            response = web.StreamResponse(status=up.status, headers=resp_headers)
            await response.prepare(request)
            async for chunk in up.content.iter_any():
                await response.write(chunk)
            await response.write_eof()
            return response

app = web.Application()
app.router.add_route(\"*\", \"/{path:.*}\", proxy)
print(f\"Auth proxy listening on 0.0.0.0:{PORT} -> {UPSTREAM_URL}\")
web.run_app(app, host=\"0.0.0.0\", port=PORT)
'"
```

## 使用方法

### 测试连接

```bash
curl http://127.0.0.1:12788/v1/chat/completions \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {"role": "user", "content": "你好"}
    ]
  }'
```

### 支持的模型

根据授权日志，可用的模型包括：

**Claude 系列**：
- `claude-opus-4.5`, `claude-opus-4.6`, `claude-opus-4.7`
- `claude-sonnet-4`, `claude-sonnet-4.5`, `claude-sonnet-4.6`
- `claude-haiku-4.5`

**GPT 系列**：
- `gpt-4o`, `gpt-4o-mini`
- `gpt-4`, `gpt-4-0613`
- `gpt-5.2`, `gpt-5.3-codex`, `gpt-5.4`
- `gpt-3.5-turbo`

**Gemini 系列**：
- `gemini-2.5-pro`
- `gemini-3-flash-preview`
- `gemini-3.1-pro-preview`

**其他**：
- `minimax-m2.5`
- `grok-code-fast-1`
- `text-embedding-3-small`

### 在应用中使用

**Python 示例（使用 OpenAI SDK）**：

```python
from openai import OpenAI

client = OpenAI(
    api_key="<YOUR_API_KEY>",
    base_url="http://your-server.com:12788/v1"
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)

print(response.choices[0].message.content)
```

**Node.js 示例**：

```javascript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: '<YOUR_API_KEY>',
  baseURL: 'http://your-server.com:12788/v1',
});

async function main() {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [{ role: 'user', content: 'Hello!' }],
  });

  console.log(completion.choices[0].message.content);
}

main();
```

## 详细技术说明

### 请求处理流程

完整的请求流程如下：

1. **客户端 → 鉴权代理 (端口 12788)**
   - 客户端发送 OpenAI 格式的请求
   - 鉴权代理提取 `Authorization: Bearer <token>` header
   - 验证 token 是否匹配环境变量 `API_KEY`
   - ❌ 不匹配：返回 `401 Unauthorized`
   - ✅ 匹配：移除 Authorization header，转发请求

2. **鉴权代理 → copilot-api (端口 9723)**
   - 构造新请求：`http://127.0.0.1:9723/v1/chat/completions`
   - 保留原始请求体和必要的 headers
   - 过滤掉 `host`, `authorization`, `transfer-encoding`, `content-length` 等冲突 headers

3. **copilot-api 处理**
   - 解析 OpenAI 格式请求
   - 使用缓存的 GitHub OAuth Token 调用 Copilot API
   - 将对话上下文传递给 GitHub Copilot 服务

4. **GitHub Copilot API**
   - 验证 OAuth Token（你的 Copilot 订阅凭证）
   - 根据模型名路由到对应的 AI 服务（OpenAI、Anthropic、Google 等）
   - 生成 AI 响应

5. **响应返回**
   - GitHub Copilot → copilot-api：转换为 OpenAI 格式
   - copilot-api → 鉴权代理：流式传输响应
   - 鉴权代理 → 客户端：保持流式输出，逐块返回

### 流式处理原理

代理使用异步流式处理，不缓存完整响应：

```python
async for chunk in up.content.iter_any():
    await response.write(chunk)
```

**优势**：
- 支持 SSE (Server-Sent Events) 流式输出
- 实时性好，适合大型响应
- 内存占用低

### 安全设计

**多层防护**：

1. **网络隔离**
   - copilot-api 仅监听 `127.0.0.1:9723`，外部无法直接访问
   - 鉴权代理作为唯一对外入口

2. **访问控制**
   - 通过 API Key 验证，防止未授权访问
   - 支持 HTTPS 加密传输（公网部署）

3. **Header 过滤**
   - 移除客户端的 Authorization header，避免泄露 API Key
   - 过滤冲突的 headers，避免协议错误

4. **最小权限原则**
   - 容器使用 `--net=host` 但不需要特殊权限
   - 证书目录只读挂载 (`:ro`)

## 网络配置说明

### 端口规划

| 服务 | 端口 | 监听地址 | 访问方式 |
|------|------|----------|----------|
| copilot-api | 9723 | 127.0.0.1 | 仅本地 |
| 鉴权代理 | 12788 | 0.0.0.0 | 对外开放 |

### 防火墙配置

**生产环境建议**：

```bash
# 仅允许来自特定 IP 的访问
iptables -A INPUT -p tcp --dport 12788 -s <TRUSTED_IP> -j ACCEPT
iptables -A INPUT -p tcp --dport 12788 -j DROP

# 禁止外部访问 9723
iptables -A INPUT -p tcp --dport 9723 -i eth0 -j DROP
```

## 故障排查

### 容器无法启动

**检查日志**：
```bash
podman logs copilot-proxy
podman logs copilot-auth-proxy
```

**常见问题**：
- 端口被占用：使用 `lsof -i :9723` 检查
- 镜像拉取失败：检查网络连接

### 授权失败

**症状**：日志中提示 "Not logged in"

**解决方法**：
1. 重新查看设备码：`podman logs copilot-proxy`
2. 访问 GitHub 授权页面重新授权
3. 确认 GitHub 账户有 Copilot 订阅

### API 返回 401

**可能原因**：
- API Key 不匹配
- Authorization header 格式错误（应为 `Bearer <key>`）

**验证方法**：
```bash
# 查看代理容器的 API_KEY 设置
podman inspect copilot-auth-proxy | grep API_KEY
```

### 响应慢或超时

**可能原因**：
- 网络延迟
- GitHub Copilot 服务负载高

**优化建议**：
- 使用流式响应 (`stream: true`)
- 增加客户端超时时间

## 监控和维护

### 查看使用情况

访问 copilot-api 提供的使用统计页面：

```
https://ericc-ch.github.io/copilot-api?endpoint=http://localhost:9723/usage
```

### 重启容器

```bash
# 重启 copilot-api
podman restart copilot-proxy

# 重启鉴权代理
podman restart copilot-auth-proxy
```

### 更新容器

```bash
# 更新 copilot-api-pro
podman exec copilot-proxy npm update -g copilot-api-pro

# 或者重新部署容器
podman rm -f copilot-proxy
# 然后重新运行部署命令
```

## 应用场景

### 1. 统一 API 接口

在多个工具中使用同一套 API 配置：
- ChatGPT 桌面客户端
- IDE 插件（如 Continue.dev）
- 自定义应用

### 2. 成本优化

- 利用现有的 GitHub Copilot 订阅（约 $10/月）
- 避免直接购买 OpenAI API 额度（按 Token 计费）
- 访问多种模型（GPT、Claude、Gemini）

### 3. 企业内部服务

- 部署私有 API 服务
- 统一管理和监控
- 控制访问权限

## 限制和注意事项

### 使用限制

- **依赖 Copilot 订阅**：需要有效的 GitHub Copilot 订阅
- **速率限制**：受 GitHub Copilot 的速率限制约束
- **模型可用性**：可用模型由 GitHub 决定，可能变化

### 合规性

- 确保使用符合 GitHub Copilot 的服务条款
- 不要用于大规模商业服务
- 注意数据隐私和安全

### 稳定性

- 这是非官方的转换方案
- copilot-api-pro 可能因 GitHub API 变化而失效
- 建议在生产环境做好备份方案

## 参考资源

- [copilot-api-pro GitHub](https://github.com/ericc-ch/copilot-api)
- [GitHub Copilot 官方文档](https://docs.github.com/en/copilot)
- [OpenAI API 文档](https://platform.openai.com/docs/api-reference)

## 总结

这个方案通过两层容器架构，实现了 GitHub Copilot 到 OpenAI API 的协议转换：

1. **copilot-api 容器**：核心转换层，处理协议适配
2. **鉴权代理容器**：安全层，提供访问控制和加密

**优势**：
- 利用 Copilot 订阅访问多种 AI 模型
- 兼容 OpenAI SDK，易于集成
- 灵活的部署选项（HTTP/HTTPS）

**适用场景**：
- 个人开发者整合多种工具
- 小团队内部 API 服务
- 成本敏感的应用场景

**建议**：
- 生产环境使用 HTTPS
- 配置防火墙限制访问
- 定期监控和更新
