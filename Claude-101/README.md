# Claude 101 — Shortened Course Summary

> Source: [Anthropic Skilljar — Claude 101](https://anthropic.skilljar.com/claude-101/383398)
> Audience: Developers, Architects, Technical Leads
> Purpose: Quick reference for Java devs adopting Claude in enterprise workflows

---

## Table of Contents

1. [What is Claude?](#1-what-is-claude)
2. [Core Capabilities](#2-core-capabilities)
3. [How Claude Works](#3-how-claude-works)
4. [Interacting with Claude](#4-interacting-with-claude)
5. [Prompt Engineering Essentials](#5-prompt-engineering-essentials)
6. [Use Cases for Developers](#6-use-cases-for-developers)
7. [Safety, Limitations & Responsible Use](#7-safety-limitations--responsible-use)
8. [Getting Started — API & Integration](#8-getting-started--api--integration)

---

## 1. What is Claude?

Claude is Anthropic's AI assistant built on large language models (LLMs). Key differentiators:

- **Constitutional AI (CAI)** — trained to be helpful, harmless, and honest
- **Long context windows** — up to 200K tokens (Claude 3.5 Sonnet / Opus)
- **Strong reasoning** — excels at code, analysis, structured output
- **Steerable** — follows system prompts and instructions precisely

---

## 2. Core Capabilities

| Capability | Description |
|---|---|
| **Text Generation** | Writing, summarization, translation |
| **Code** | Generation, review, debugging, refactoring (Java, Python, TS, etc.) |
| **Analysis** | Data interpretation, document analysis, reasoning chains |
| **Vision** | Image understanding (diagrams, screenshots, charts) |
| **Structured Output** | JSON, XML, YAML, tabular data on demand |
| **Multi-turn Conversation** | Context retention across messages |

---

## 3. How Claude Works

### Architecture (High Level)
```
User Prompt → Tokenization → Transformer Model → Output Tokens → Response
```

### Key Concepts

- **Tokens** — subword units (~4 chars in English). Pricing & limits are token-based.
- **Context Window** — the "working memory" for a single conversation (input + output combined).
- **Temperature** — controls randomness (0 = deterministic, 1 = creative).
- **System Prompt** — sets persona, rules, constraints before user messages.

### Model Tiers

| Model | Best For |
|---|---|
| **Claude 3.5 Sonnet** | Balanced speed + intelligence; ideal for most dev tasks |
| **Claude 3 Opus** | Hardest reasoning tasks, long-form analysis |
| **Claude 3 Haiku** | Speed-critical, high-throughput, simpler tasks |

---

## 4. Interacting with Claude

### Channels
- **claude.ai** — web chat UI
- **API (Messages API)** — programmatic access (REST)
- **Amazon Bedrock / Google Vertex AI** — managed cloud endpoints
- **Anthropic Console** — playground, prompt testing, usage dashboard

### Messages API Structure (simplified)

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1024,
  "system": "You are a senior Java architect...",
  "messages": [
    {"role": "user", "content": "Review this Spring Boot controller..."}
  ]
}
```

---

## 5. Prompt Engineering Essentials

### The CRAFT Framework

| Letter | Meaning | Example |
|---|---|---|
| **C** | Context | "You are reviewing a Java 21 microservice..." |
| **R** | Role | "Act as a senior architect" |
| **A** | Action | "Identify potential thread-safety issues" |
| **F** | Format | "Return findings as a markdown table" |
| **T** | Tone / Constraints | "Be concise. No code rewrites unless asked." |

### Best Practices

1. **Be specific** — vague prompts → vague answers
2. **Provide examples** (few-shot) — show input/output pairs
3. **Use XML tags** for structure:
   ```
   <code>...</code>
   <instructions>...</instructions>
   ```
4. **Chain of thought** — ask Claude to "think step by step" for complex reasoning
5. **Iterative refinement** — treat prompts as code; version and test them
6. **Constrain output** — "Respond ONLY with JSON. No explanation."

### Anti-patterns
- ❌ Asking multiple unrelated questions in one prompt
- ❌ No context about tech stack or constraints
- ❌ Expecting real-time/internet data (Claude has a training cutoff)

---

## 6. Use Cases for Developers

### Code-Centric
- **Code review** — security, performance, style
- **Refactoring** — legacy Java → modern idioms (records, sealed classes, pattern matching)
- **Test generation** — JUnit 5, Mockito, integration tests
- **Documentation** — Javadoc, ADRs, README generation
- **Debugging** — stack trace analysis, root cause hypotheses

### Architecture & Design
- **Design review** — evaluate diagrams, propose alternatives
- **API design** — OpenAPI spec generation/critique
- **Migration planning** — monolith → microservices, Java 8 → 21

### Productivity
- **Summarize PRs / RFCs** — extract key decisions
- **Generate boilerplate** — DTOs, mappers, configs
- **Explain complex code** — unfamiliar codebases, legacy systems

---

## 7. Safety, Limitations & Responsible Use

### Limitations
| Limitation | Implication |
|---|---|
| **Knowledge cutoff** | No awareness of events after training data date |
| **No internet access** | Cannot fetch URLs, call APIs, or browse |
| **Hallucinations** | May generate plausible but incorrect info — always verify |
| **No state** | Each API call is stateless; context must be resent |
| **Token limits** | Very large codebases may need chunking strategies |

### Responsible Use Guidelines
- **Don't trust blindly** — validate generated code (tests, reviews)
- **Don't share secrets** — no API keys, passwords, PII in prompts (unless using zero-retention API)
- **Understand data policies** — Anthropic's API has data retention policies; read them
- **Human in the loop** — Claude augments, not replaces, engineering judgment
- **Bias awareness** — LLMs can reflect biases; critically evaluate recommendations

---

## 8. Getting Started — API & Integration

### Quick Start (Java / HTTP)

```java
// Using Java HttpClient (Java 11+)
HttpClient client = HttpClient.newHttpClient();

String body = """
    {
      "model": "claude-sonnet-4-20250514",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude!"}]
    }
    """;

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.anthropic.com/v1/messages"))
    .header("x-api-key", System.getenv("ANTHROPIC_API_KEY"))
    .header("anthropic-version", "2023-06-01")
    .header("content-type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString(body))
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());
```

### SDK Options
- **Python SDK** — `anthropic` pip package (official)
- **TypeScript SDK** — `@anthropic-ai/sdk` (official)
- **Java** — community wrappers or direct HTTP (no official Java SDK yet)
- **Spring AI** — has Anthropic integration for Spring Boot apps

### Enterprise Integration Patterns
```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Your App   │────▶│  API Gateway │────▶│  Anthropic API  │
│(Spring Boot)│     │ (rate limit, │     │  or Bedrock     │
└─────────────┘     │  auth, log)  │     └─────────────────┘
                    └──────────────┘
```

- Use **API Gateway** for rate limiting, key rotation, audit logging
- Consider **Amazon Bedrock** for VPC-level isolation and AWS IAM auth
- Implement **retry with exponential backoff** for 429/5xx errors
- Cache responses where deterministic output is acceptable

---

## Key Takeaways (TL;DR)

1. Claude is a powerful reasoning engine — treat it like a very fast, knowledgeable (but fallible) pair programmer.
2. Prompt quality → output quality. Invest in prompt engineering like you invest in code quality.
3. Always verify — especially for security-sensitive code or architectural decisions.
4. Use structured prompts (system + XML tags + examples) for consistent, production-grade outputs.
5. Start with Sonnet for daily dev work; escalate to Opus for hard problems.

---

*Last updated: 2025-05-01*
