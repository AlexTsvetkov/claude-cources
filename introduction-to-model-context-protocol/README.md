# Introduction to Model Context Protocol (MCP) — Shortened Course Summary

> Source: [Anthropic Skilljar — Introduction to Model Context Protocol](https://anthropic.skilljar.com/introduction-to-model-context-protocol)
> Audience: Developers, Architects, Technical Leads
> Purpose: Understanding and implementing MCP for connecting AI to external tools and data

---

## Table of Contents

1. [What is MCP?](#1-what-is-mcp)
2. [Architecture & Components](#2-architecture--components)
3. [How MCP Works](#3-how-mcp-works)
4. [MCP Primitives](#4-mcp-primitives)
5. [Building an MCP Server](#5-building-an-mcp-server)
6. [MCP Clients & Hosts](#6-mcp-clients--hosts)
7. [Transport Layer](#7-transport-layer)
8. [Security Considerations](#8-security-considerations)
9. [Real-World Use Cases](#9-real-world-use-cases)
10. [Key Takeaways](#10-key-takeaways)

---

## 1. What is MCP?

The **Model Context Protocol (MCP)** is an open standard that provides a universal way to connect AI models to external data sources, tools, and services.

### The Problem MCP Solves

Before MCP:
- Every AI integration required custom code
- N models x M tools = N*M integrations
- No standard way to expose capabilities to AI

With MCP:
- One standard protocol
- N models + M tools = N+M integrations
- Any MCP client talks to any MCP server
- Plug-and-play tool ecosystem

### Analogy

MCP is to AI tools what **USB is to peripherals** — a universal connector standard.

---

## 2. Architecture & Components

### Three-Layer Architecture

```
MCP HOST (Claude Desktop, IDE, Custom App)
  └── MCP Client (one per server connection)
        └── MCP Server (exposes tools/resources)
```

### Components

| Component | Role | Example |
|---|---|---|
| **Host** | Application that embeds AI | Claude Desktop, VS Code, your Spring app |
| **Client** | Protocol connector inside the host | Manages connection to one server |
| **Server** | Exposes tools/resources to AI | Database server, GitHub server, Jira server |

### Key Design Principles
- **Servers are lightweight** — focused on specific capabilities
- **Clients manage connections** — one client per server
- **Hosts orchestrate** — the application that brings it all together
- **Stateful sessions** — maintain context across interactions

---

## 3. How MCP Works

### Communication Flow

```
1. Host starts MCP Server (subprocess or remote)
2. Client performs capability negotiation (initialize handshake)
3. Client discovers available tools/resources/prompts
4. User asks AI a question
5. AI decides to use a tool -> sends tool call via MCP
6. Server executes tool, returns result
7. AI uses result to form response
```

### Protocol Messages

MCP uses **JSON-RPC 2.0** for communication:

```json
// Request (Client -> Server)
{"jsonrpc": "2.0", "id": 1, "method": "tools/call",
 "params": {"name": "query_database", "arguments": {"sql": "SELECT ..."}}}

// Response (Server -> Client)
{"jsonrpc": "2.0", "id": 1,
 "result": {"content": [{"type": "text", "text": "Found 42 results..."}]}}
```

### Lifecycle

```
Initialize -> Initialized -> Normal Operation -> Shutdown
```

---

## 4. MCP Primitives

MCP servers expose three types of primitives:

### Tools (Model-Controlled)

Functions that the AI model can invoke. Like POST endpoints.

```json
{
  "name": "create_jira_ticket",
  "description": "Create a new Jira ticket in a specified project",
  "inputSchema": {
    "type": "object",
    "properties": {
      "project": {"type": "string"},
      "title": {"type": "string"},
      "priority": {"type": "string", "enum": ["Critical","High","Medium","Low"]}
    },
    "required": ["project", "title"]
  }
}
```

### Resources (Application-Controlled)

Data sources that provide context. Like GET endpoints.

```json
{"uri": "db://orders/schema", "name": "Orders Table Schema", "mimeType": "application/json"}
```

### Prompts (User-Controlled)

Reusable prompt templates.

```json
{
  "name": "code_review",
  "description": "Review code for a specific file",
  "arguments": [
    {"name": "file_path", "required": true},
    {"name": "focus", "description": "security, performance, or style"}
  ]
}
```

### Comparison

| Primitive | Controlled By | Analogy | Example |
|---|---|---|---|
| **Tools** | AI Model | POST endpoints | create_ticket, run_query |
| **Resources** | Application | GET endpoints | file contents, DB schema |
| **Prompts** | User | Saved templates | code_review, debug_error |

---

## 5. Building an MCP Server

### TypeScript (Official SDK)

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({ name: "order-service-mcp", version: "1.0.0" });

server.tool(
  "get_order_status",
  "Get the current status of an order by ID",
  { order_id: { type: "string", description: "Order ID" } },
  async ({ order_id }) => {
    const status = await orderService.getStatus(order_id);
    return { content: [{ type: "text", text: JSON.stringify(status) }] };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Python

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server

app = Server("order-service-mcp")

@app.tool()
async def get_order_status(order_id: str) -> str:
    status = await order_service.get_status(order_id)
    return json.dumps(status)

async def main():
    async with stdio_server() as (read, write):
        await app.run(read, write)
```

### Java (Spring Boot Pattern)

```java
public class McpServer {
    private final ObjectMapper mapper = new ObjectMapper();
    private final Map<String, ToolHandler> tools = new HashMap<>();

    public void registerTool(String name, String desc, ToolHandler handler) {
        tools.put(name, handler);
    }

    public void start() throws IOException {
        var stdin = new BufferedReader(new InputStreamReader(System.in));
        String line;
        while ((line = stdin.readLine()) != null) {
            var request = mapper.readValue(line, JsonRpcRequest.class);
            var response = switch (request.method()) {
                case "initialize" -> handleInitialize(request);
                case "tools/list" -> handleToolsList(request);
                case "tools/call" -> handleToolCall(request);
                default -> errorResponse(request.id(), "Method not found");
            };
            System.out.println(mapper.writeValueAsString(response));
            System.out.flush();
        }
    }
}
```

---

## 6. MCP Clients & Hosts

### Existing MCP Hosts

| Host | Description |
|---|---|
| **Claude Desktop** | Built-in MCP support |
| **Claude Code** | CLI with MCP server access |
| **VS Code (Copilot)** | Editor integration |
| **Continue.dev** | Open-source AI coding assistant |
| **Custom Apps** | Your own applications |

### Configuring Claude Desktop

```json
// ~/Library/Application Support/Claude/claude_desktop_config.json
{
  "mcpServers": {
    "database": {
      "command": "node",
      "args": ["/path/to/db-mcp-server/index.js"],
      "env": {"DB_URL": "postgresql://localhost:5432/mydb"}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {"GITHUB_TOKEN": "ghp_..."}
    }
  }
}
```

### Java MCP Client Pattern

```java
public class McpClient {
    private Process serverProcess;
    private BufferedReader reader;
    private BufferedWriter writer;
    private int requestId = 0;

    public McpClient(String command, String... args) throws IOException {
        var pb = new ProcessBuilder(command);
        pb.command().addAll(List.of(args));
        serverProcess = pb.start();
        reader = new BufferedReader(new InputStreamReader(serverProcess.getInputStream()));
        writer = new BufferedWriter(new OutputStreamWriter(serverProcess.getOutputStream()));
    }

    public JsonNode callTool(String name, Map<String, Object> args) throws IOException {
        var req = Map.of("jsonrpc","2.0","id",++requestId,
            "method","tools/call","params",Map.of("name",name,"arguments",args));
        writer.write(new ObjectMapper().writeValueAsString(req));
        writer.newLine(); writer.flush();
        return new ObjectMapper().readTree(reader.readLine());
    }
}
```

---

## 7. Transport Layer

| Transport | Best For | Pros | Cons |
|---|---|---|---|
| **stdio** | Local servers, CLI | Simple, secure, no network | Local only |
| **HTTP + SSE** | Remote/shared servers | Network accessible, multi-client | Complex setup |
| **Streamable HTTP** | Cloud services | Modern, simpler than SSE | Newest, less tooling |

### Choosing Transport

| Scenario | Transport |
|---|---|
| Local dev tools (Claude Desktop) | stdio |
| Shared team server | HTTP + SSE |
| Cloud-hosted MCP service | Streamable HTTP |
| CI/CD pipelines | stdio |

---

## 8. Security Considerations

### Threat Model & Mitigations

| Threat | Mitigation |
|---|---|
| Prompt injection via tool results | Sanitize all outputs |
| Unauthorized tool execution | Permission system, user confirmation |
| Data exfiltration | Restrict capabilities, audit logging |
| Secrets exposure | Env vars, never in tool responses |
| Server compromise | Least privilege, sandboxing |

### Best Practices

1. **Least privilege** — servers only access what they need
2. **Input validation** — validate all tool arguments server-side
3. **Output sanitization** — clean tool results before returning
4. **Authentication** — use tokens/keys for remote servers
5. **Audit logging** — log all tool invocations
6. **User confirmation** — require approval for destructive actions
7. **Sandboxing** — run servers with minimal OS permissions

---

## 9. Real-World Use Cases

### For Java/Spring Teams

| Use Case | MCP Server | What It Does |
|---|---|---|
| **Database queries** | postgres-mcp | AI queries your DB safely (read-only) |
| **Git operations** | github-mcp | Create PRs, review code, manage issues |
| **Jira/Project mgmt** | jira-mcp | Create tickets, update status, query backlog |
| **Monitoring** | grafana-mcp | Query metrics, check alerts |
| **Documentation** | confluence-mcp | Search/read wiki, update docs |
| **CI/CD** | jenkins-mcp | Trigger builds, check status |
| **Custom services** | your-service-mcp | Expose your microservice APIs to AI |

### Example: Custom Spring Boot MCP Server

Expose your order service to AI:

```
Tools:
- get_order(order_id) -> order details
- search_orders(criteria) -> matching orders
- get_order_metrics(date_range) -> KPIs

Resources:
- orders://schema -> DB schema
- orders://api-docs -> OpenAPI spec
```

### Available Community Servers

- `@modelcontextprotocol/server-github` — GitHub operations
- `@modelcontextprotocol/server-postgres` — PostgreSQL queries
- `@modelcontextprotocol/server-filesystem` — File system access
- `@modelcontextprotocol/server-slack` — Slack messaging
- `@modelcontextprotocol/server-puppeteer` — Browser automation

---

## 10. Key Takeaways

### TL;DR for the Java Architect

1. **MCP = USB for AI tools** — universal standard connecting AI to any service
2. **Three primitives** — Tools (actions), Resources (data), Prompts (templates)
3. **JSON-RPC 2.0** — simple, well-known protocol underneath
4. **Build servers in any language** — TS, Python, Java all supported
5. **Security-first** — validate inputs, sanitize outputs, audit everything
6. **Start with existing servers** — GitHub, Postgres, filesystem already available
7. **Build custom servers** — expose your Spring Boot services to AI

### Architecture Decision: When to Use MCP

| Use MCP When | Don't Use MCP When |
|---|---|
| AI needs access to live data | Static/hardcoded data suffices |
| Multiple AI tools need same data source | One-off integration |
| Team shares AI tool configurations | Single developer, simple use case |
| You want plug-and-play tool swapping | Tightly coupled, custom AI pipeline |
| Security/audit requirements exist | Prototyping/POC phase |

### Quick Start

```bash
# 1. Install an existing MCP server
npx -y @modelcontextprotocol/server-github

# 2. Configure in Claude Desktop config
# 3. Ask Claude: "What are my open PRs?"
# 4. Claude uses MCP to query GitHub and responds
```

---

*Last updated: 2025-05-01*
