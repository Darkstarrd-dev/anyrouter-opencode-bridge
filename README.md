# AnyRouter OpenCode Bridge

一个专门为 [OpenCode](https://github.com/anomalyco/opencode) 和 [CherryStudio](https://cherry-ai.com) 设计的本地代理桥接工具，用于解决接入 [AnyRouter](https://anyrouter.top) (Claude Code API) 时遇到的 WAF 拦截和 TLS 指纹问题。

## 背景

AnyRouter 是一个专为 Claude Code 设计的 API Provider，具有严格的安全防护：
1.  **TLS 指纹校验**：拦截普通 Python/Node.js 的 HTTP 请求，必须使用 HTTP/2 且具备特定指纹。
2.  **严格的 Header 检查**：强制校验 `anthropic-client-name` 等客户端标识。
3.  **请求体校验**：检测请求是否包含 Claude Code 的工具定义。

本项目通过运行一个轻量级的本地 Python 代理服务器（FastAPI + httpx/HTTP2），作为中间人进行协议清洗和伪装，完美解决上述问题。

## 功能特性

*   ✅ **HTTP/2 支持**：使用 `httpx` 绕过 TLS 指纹检测。
*   ✅ **Header 伪装**：自动注入 Claude Code 的真实 Header（通过 mitmproxy 抓包验证）。
*   ✅ **工具注入**：对所有 Claude 模型自动注入 Claude Code 工具定义，绕过服务端检测。
*   ✅ **Body 清洗**：过滤可能导致 WAF 拦截的字段。
*   ✅ **流式透传**：完美支持 SSE (Server-Sent Events) 流式响应，打字机效果流畅。
*   ✅ **连接保持**：内置连接池和重试机制，应对上游不稳定性。
*   ✅ **配置灵活**：支持交互式配置向导、JSON 配置文件和热重载。

## 支持的客户端

| 客户端 | 支持状态 | 推荐协议 |
|--------|----------|----------|
| OpenCode | ✅ 完全支持 | Anthropic (@ai-sdk/anthropic) |
| CherryStudio | ✅ 完全支持 | Anthropic |

## 快速开始

### 1. 下载

从 [Releases](https://github.com/Darkstarrd-dev/anyrouter-opencode-bridge/releases) 下载最新版本的 zip 包并解压。

### 2. 安装依赖

需要 Python 3.8+ 环境。

```bash
pip install -r requirements.txt
```

### 3. 运行与配置

首次运行会自动进入配置向导：

```bash
python main.py
```

按提示输入：
*   **API Key**: 你的 AnyRouter API Key (`sk-...`)
*   **Proxy**: 是否使用系统代理（如 Clash/v2ray，建议开启以提高连接稳定性）

配置完成后，服务将在 `http://127.0.0.1:8765` 启动。

---

## 配置 OpenCode

在 OpenCode 的配置文件 (`~/.config/opencode/opencode.json`) 中添加或修改 Provider 配置：

```json
"anyrouter": {
  "npm": "@ai-sdk/anthropic",
  "name": "AnyRouter (via Bridge)",
  "options": {
    "baseURL": "http://127.0.0.1:8765/v1",
    "apiKey": "sk-placeholder" 
  },
  "models": {
    "claude-haiku-4-5-20251001": {
      "name": "Claude Haiku 4.5",
      "limit": { "context": 256000, "output": 128000 }
    },
    "claude-sonnet-4-5-20250929": {
      "name": "Claude Sonnet 4.5",
      "limit": { "context": 256000, "output": 128000 }
    },
    "claude-opus-4-5-20251101": {
      "name": "Claude Opus 4.5",
      "limit": { "context": 256000, "output": 128000 }
    }
  }
}
```

---

## 配置 CherryStudio

CherryStudio 请务必使用 **Anthropic** 协议接入。

### 配置步骤

1. 打开 CherryStudio 设置
2. 添加新的 API Provider，选择 **Anthropic** 类型
3. 配置如下：

| 配置项 | 值 |
|--------|-----|
| API 地址 | `http://127.0.0.1:8765/v1` |
| API Key | 任意值（如 `sk-placeholder`） |

### 可用模型

在模型列表中添加以下模型：

- `claude-haiku-4-5-20251001`
- `claude-sonnet-4-5-20250929`
- `claude-opus-4-5-20251101`

---

## 常见问题

**Q: 为什么 CherryStudio 提示 404？**
A: 请确保：
1. 代理服务器正在运行。
2. 选择了 **Anthropic** 协议而不是 OpenAI 协议。
3. API 地址填的是 `http://127.0.0.1:8765/v1`。

**Q: 需要一直开着终端吗？**
A: 是的，或者使用 `nohup` / `pm2` / `nssm` (Windows) 将其作为后台服务运行。

---

## 许可证

MIT
