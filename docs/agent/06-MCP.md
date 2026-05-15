# MCP（Model Context Protocol）

> 目标：理解 MCP 是什么、解决什么问题、架构如何、如何接入。

---

## 一、是什么？

**MCP** 是 Anthropic 提出的开放标准协议，定义了 LLM 与外部工具/数据源之间的统一通信方式。

**类比**：MCP 之于 AI 工具调用，就像 USB 之于外设——统一接口，插上就用，不用为每个设备单独写驱动。

**解决的问题**：以前每个 AI 应用都要单独集成每个工具（N×M 的问题），MCP 让工具只需实现一次，所有支持 MCP 的 AI 应用都能用（N+M）。

---

## 二、架构

```
┌─────────────────────────────┐
│        MCP Host             │  ← 你的 AI 应用（Claude Desktop / Cursor / 自研 Agent）
│  ┌──────────────────────┐   │
│  │    MCP Client        │   │  ← 内嵌在 Host 中，管理与 Server 的连接
│  └──────────┬───────────┘   │
└─────────────┼───────────────┘
              │ MCP 协议（JSON-RPC over stdio / SSE）
   ┌──────────┴──────────┐
   │                     │
┌──▼──────┐         ┌────▼────┐
│MCP Server│         │MCP Server│  ← 每个 Server 暴露一组工具/资源
│(文件系统)│         │(数据库)  │
└─────────┘         └─────────┘
```

---

## 三、核心概念

| 概念 | 说明 |
|------|------|
| **Tools** | 模型可调用的函数，如"查询数据库"、"执行代码" |
| **Resources** | 模型可读取的数据，如文件内容、API 响应 |
| **Prompts** | 预定义的提示模板，由 Server 提供 |
| **Sampling** | Server 请求 Host 执行 LLM 推理（反向调用） |

---

## 四、传输方式

| 方式 | 适用场景 |
|------|---------|
| **stdio** | 本地工具，Server 作为子进程运行，最常用 |
| **SSE（HTTP）** | 远程工具，支持跨网络部署 |

---

## 五、快速接入示例（Python SDK）

### 实现一个 MCP Server

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

app = Server("expense-tools")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="get_expense",
            description="根据 ID 查询报销单详情",
            inputSchema={
                "type": "object",
                "properties": {
                    "expense_id": {"type": "string"}
                },
                "required": ["expense_id"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "get_expense":
        result = db.query_expense(arguments["expense_id"])
        return [TextContent(type="text", text=str(result))]

async def main():
    async with stdio_server() as (read, write):
        await app.run(read, write, app.create_initialization_options())
```

### 在 Claude Desktop 中配置

```json
{
  "mcpServers": {
    "expense-tools": {
      "command": "python",
      "args": ["/path/to/server.py"]
    }
  }
}
```

---

## 六、MCP vs Function Call

| | Function Call | MCP |
|--|--------------|-----|
| **标准化程度** | 各家 API 格式不同 | 统一开放标准 |
| **工具复用** | 绑定在单个应用 | 跨应用共享 |
| **工具发现** | 手动在代码里写 | Server 动态声明 |
| **适用阶段** | 快速开发单个功能 | 工具生态规模化 |

---

## 参考资料

- [MCP 官网](https://modelcontextprotocol.io/)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP Server 市场](https://github.com/modelcontextprotocol/servers)
