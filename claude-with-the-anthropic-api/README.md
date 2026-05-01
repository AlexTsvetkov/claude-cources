# Claude with the Anthropic API — Shortened Course Summary

> Source: [Anthropic Skilljar — Claude with the Anthropic API](https://anthropic.skilljar.com/claude-with-the-anthropic-api)
> Audience: Developers, Architects, Technical Leads
> Purpose: Practical guide to integrating Claude via the Anthropic Messages API

---

## Table of Contents

1. [API Overview](#1-api-overview)
2. [Authentication & Setup](#2-authentication--setup)
3. [Messages API — Core Endpoint](#3-messages-api--core-endpoint)
4. [System Prompts & Roles](#4-system-prompts--roles)
5. [Multi-turn Conversations](#5-multi-turn-conversations)
6. [Streaming Responses](#6-streaming-responses)
7. [Vision (Image Inputs)](#7-vision-image-inputs)
8. [Tool Use (Function Calling)](#8-tool-use-function-calling)
9. [Model Selection & Parameters](#9-model-selection--parameters)
10. [Error Handling & Best Practices](#10-error-handling--best-practices)
11. [Enterprise Considerations](#11-enterprise-considerations)

---

## 1. API Overview

The Anthropic API provides programmatic access to Claude models via a REST interface.

```
Client (Java) --POST--> api.anthropic.com/v1/messages --> Claude Model
             <--JSON---                              <--
```

**Key characteristics:**
- RESTful JSON API
- Stateless — no server-side conversation memory
- Token-based pricing (input + output tokens)
- Rate-limited per organization

---

## 2. Authentication & Setup

### API Key
```bash
export ANTHROPIC_API_KEY="sk-ant-api03-..."
```

### Required Headers

| Header | Value |
|---|---|
| x-api-key | Your API key |
| anthropic-version | 2023-06-01 (latest stable) |
| content-type | application/json |

### Java Setup (HttpClient)

```java
private static final HttpClient HTTP_CLIENT = HttpClient.newHttpClient();
private static final String API_KEY = System.getenv("ANTHROPIC_API_KEY");
private static final String API_URL = "https://api.anthropic.com/v1/messages";
private static final String API_VERSION = "2023-06-01";

private HttpRequest.Builder baseRequest() {
    return HttpRequest.newBuilder()
        .uri(URI.create(API_URL))
        .header("x-api-key", API_KEY)
        .header("anthropic-version", API_VERSION)
        .header("content-type", "application/json");
}
```

### Using Spring AI (Spring Boot)

```yaml
spring:
  ai:
    anthropic:
      api-key: \${ANTHROPIC_API_KEY}
      chat:
        options:
          model: claude-sonnet-4-20250514
          max-tokens: 4096
```

---

## 3. Messages API — Core Endpoint

### Request Structure

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1024,
  "system": "You are a helpful assistant.",
  "messages": [
    {"role": "user", "content": "Explain dependency injection in Spring."}
  ]
}
```

### Response Structure

```json
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "content": [{"type": "text", "text": "Dependency injection (DI) in Spring is..."}],
  "model": "claude-sonnet-4-20250514",
  "stop_reason": "end_turn",
  "usage": {"input_tokens": 25, "output_tokens": 150}
}
```

### Key Fields

| Field | Purpose |
|---|---|
| model | Which Claude model to use |
| max_tokens | Maximum output tokens (required) |
| system | System prompt — sets behavior/persona |
| messages | Array of conversation turns |
| stop_reason | Why generation stopped: end_turn, max_tokens, stop_sequence |
| usage | Token counts for billing/monitoring |

---

## 4. System Prompts & Roles

### System Prompt Purpose
- Sets Claude persona, expertise, and constraints
- Applied before user messages
- Not visible to end users in the conversation

### Message Roles

| Role | Purpose |
|---|---|
| system | Top-level field; sets overall behavior |
| user | Human input / questions |
| assistant | Claude responses (also used for pre-filling) |

### Pre-filling (Assistant Prefill Technique)

Force Claude to start its response a certain way:

```json
{
  "messages": [
    {"role": "user", "content": "List 3 design patterns. Respond as JSON."},
    {"role": "assistant", "content": "{"}
  ]
}
```

Claude will continue from { — ensuring JSON output.

---

## 5. Multi-turn Conversations

Since the API is **stateless**, you must send the full conversation history each time:

```json
{
  "messages": [
    {"role": "user", "content": "What is the Builder pattern?"},
    {"role": "assistant", "content": "The Builder pattern separates..."},
    {"role": "user", "content": "Show me a Java example with Lombok."}
  ]
}
```

### Best Practices
- **Trim old messages** when approaching context limits
- **Summarize** long conversations periodically
- **Cache** repeated system prompts (save tokens on input)

---

## 6. Streaming Responses

For real-time UX, use Server-Sent Events (SSE):

```json
{"model": "claude-sonnet-4-20250514", "max_tokens": 1024, "stream": true, "messages": [...]}
```

### SSE Events

| Event | Purpose |
|---|---|
| message_start | Message metadata |
| content_block_delta | Incremental text chunks |
| message_stop | Generation complete |

### Java Streaming

```java
HTTP_CLIENT.sendAsync(request, HttpResponse.BodyHandlers.ofLines())
    .thenAccept(response -> {
        response.body().forEach(line -> {
            if (line.startsWith("data: ")) {
                processStreamEvent(line.substring(6));
            }
        });
    });
```

---

## 7. Vision (Image Inputs)

Claude can process images alongside text:

```json
{
  "messages": [{
    "role": "user",
    "content": [
      {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": "<base64>"}},
      {"type": "text", "text": "What architecture pattern is shown in this diagram?"}
    ]
  }]
}
```

### Dev Use Cases for Vision
- Analyze architecture diagrams
- Read error screenshots
- Interpret monitoring dashboards
- Reverse-engineer UI mockups to code

---

## 8. Tool Use (Function Calling)

Claude can invoke tools/functions you define, enabling agentic workflows.

### Tool Use Flow

```
1. User asks question
2. Claude decides to call a tool -> returns tool_use content block
3. Your code executes the tool
4. You send tool result back as tool_result message
5. Claude formulates final answer using tool result
```

### Defining Tools

```json
{
  "tools": [{
    "name": "get_service_health",
    "description": "Check the health status of a microservice",
    "input_schema": {
      "type": "object",
      "properties": {
        "service_name": {"type": "string"},
        "environment": {"type": "string", "enum": ["dev", "staging", "prod"]}
      },
      "required": ["service_name", "environment"]
    }
  }]
}
```

### Java Tool Execution Pattern

```java
public class ToolExecutor {
    private final Map<String, Function<JsonNode, String>> tools = Map.of(
        "get_service_health", this::getServiceHealth,
        "run_query", this::runQuery
    );

    public String execute(String toolName, JsonNode input) {
        var handler = tools.get(toolName);
        if (handler == null) throw new IllegalArgumentException("Unknown tool: " + toolName);
        return handler.apply(input);
    }
}
```

---

## 9. Model Selection & Parameters

### Models

| Model | ID | Best For |
|---|---|---|
| Claude 3.5 Sonnet | claude-sonnet-4-20250514 | Daily dev work, balanced |
| Claude 3 Opus | claude-3-opus-20240229 | Complex reasoning |
| Claude 3.5 Haiku | claude-3-5-haiku-20241022 | High-throughput, simple tasks |

### Parameters

| Parameter | Type | Purpose |
|---|---|---|
| max_tokens | int | Max output tokens (required) |
| temperature | float | Randomness (0=deterministic, 1=creative) |
| top_p | float | Nucleus sampling |
| stop_sequences | string[] | Custom stop strings |
| stream | bool | Enable SSE streaming |

### Recommendations for Dev Tasks

| Task | Temperature | Max Tokens |
|---|---|---|
| Code generation | 0.0-0.2 | 4096 |
| Code review | 0.0 | 2048 |
| JSON/structured output | 0.0 | 1024 |
| Brainstorming | 0.8-1.0 | 2048 |

---

## 10. Error Handling & Best Practices

### Common HTTP Errors

| Code | Meaning | Action |
|---|---|---|
| 400 | Invalid request | Fix request body |
| 401 | Auth failed | Check API key |
| 429 | Rate limited | Backoff and retry |
| 500 | Server error | Retry with backoff |
| 529 | Overloaded | Retry with longer backoff |

### Retry Strategy (Java)

```java
public String sendWithRetry(String body) throws Exception {
    for (int attempt = 0; attempt < 3; attempt++) {
        var response = HTTP_CLIENT.send(buildRequest(body), BodyHandlers.ofString());
        if (response.statusCode() == 200) return response.body();
        if (response.statusCode() == 429 || response.statusCode() >= 500) {
            Thread.sleep((long) Math.pow(2, attempt) * 1000);
            continue;
        }
        throw new RuntimeException("API error: " + response.statusCode());
    }
    throw new RuntimeException("Max retries exceeded");
}
```

### Best Practices
- Always set max_tokens explicitly
- Use temperature 0 for deterministic code output
- Monitor usage.input_tokens + usage.output_tokens for cost control
- Implement circuit breakers for production systems
- Log request IDs for debugging with Anthropic support

---

## 11. Enterprise Considerations

### Deployment Options

| Option | Best For |
|---|---|
| Direct API | Full control, any cloud |
| Amazon Bedrock | AWS-native, VPC isolation, IAM auth |
| Google Vertex AI | GCP-native, managed endpoints |

### Security Checklist
- [ ] API keys in secrets manager (not code/env files)
- [ ] Network: restrict egress to Anthropic IPs
- [ ] Audit logging on all API calls
- [ ] PII detection before sending to API
- [ ] Data retention policy aligned with Anthropic ToS

### Architecture Pattern

```
App -> API Gateway (auth, rate limit, log) -> Anthropic API / Bedrock
                                           -> Response cache (optional)
```

---

## Key Takeaways

1. **Messages API is stateless** — manage conversation history client-side
2. **System prompts are powerful** — invest time in crafting them
3. **Tool use enables agents** — Claude can call your functions
4. **Streaming for UX** — use SSE for real-time responses
5. **Temperature 0 for code** — deterministic output for dev tasks
6. **Always handle errors** — implement retry with exponential backoff
7. **Vision unlocks diagrams** — feed architecture images directly

---

*Last updated: 2025-05-01*
