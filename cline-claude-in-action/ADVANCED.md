# Cline + Claude in Action — Advanced Topics

[← Back to Main Course](README.md)

---

## 9. MCP Servers in Cline

### What is MCP?

The Model Context Protocol (MCP) allows Cline to connect to external tools and data sources, extending Claude's capabilities beyond the file system.

### Configuring MCP Servers

MCP servers are configured in Cline's settings via JSON:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxxxxxxxxxxx"
      }
    }
  }
}
```

### Popular MCP Servers

| Server | Purpose | Example Use Case |
|--------|---------|------------------|
| **postgres/sqlite** | Database access | "Show me the users table schema" |
| **github** | GitHub API | "Create an issue for this bug" |
| **brave-search** | Web search | "Find docs for this library" |
| **memory** | Persistent memory | Remember decisions across sessions |
| **puppeteer** | Browser automation | "Test the login flow" |
| **filesystem** | Extended file access | Access files outside workspace |

### Using MCP in Practice

Once configured, use naturally in prompts:

```
Query the database to show me all tables and their relationships,
then generate an ERD diagram in Mermaid format.
```

---

## 10. Advanced Workflows

### 10.1 Test-Driven Development

```
Let's do TDD for a new PasswordValidator utility:
1. Write failing tests first for these rules:
   - Minimum 8 characters
   - At least one uppercase letter
   - At least one number
   - At least one special character
2. Then implement the minimum code to pass all tests.
3. Finally, refactor for clean code.
Show me the test output at each step.
```

### 10.2 Debugging Sessions

```
The application crashes with "JavaScript heap out of memory" 
when processing large files. Please:
1. Investigate the file processing pipeline in src/processors/
2. Identify where the memory issue occurs
3. Implement streaming/chunked processing to fix it
4. Add a test with a large mock file to prevent regression
```

### 10.3 Code Migration

```
Migrate our Express.js API from JavaScript to TypeScript:
- Convert one file at a time, starting with utils/
- Add proper type definitions (no `any` types)
- Update imports to use .ts extensions
- Ensure all existing tests pass after each file conversion
Start with src/utils/validators.js
```

### 10.4 Documentation Generation

```
Generate comprehensive documentation for this project:
1. Create docs/ directory
2. Generate API.md with all endpoint documentation
3. Create ARCHITECTURE.md explaining the system design
4. Add CONTRIBUTING.md with setup instructions
Use information from the existing codebase only.
```

### 10.5 Security Audit

```
Perform a security audit of this codebase. Check for:
- SQL injection vulnerabilities
- XSS vulnerabilities
- Missing input validation
- Hardcoded secrets
- Insecure dependencies
Output a table: Location | Issue | Severity | Fix Recommendation
Don't make changes yet, just report findings.
```

### 10.6 Multi-File Refactoring

```
Standardize error handling across the codebase:
1. Create AppError class hierarchy (ValidationError, NotFoundError, etc.)
2. Create error handling middleware
3. Update ALL route handlers to use new error classes
4. Update tests to verify error responses
```

---

## 11. Cost Management & Model Selection

### Understanding Costs

Cline shows real-time cost tracking:

```
Task Cost: $0.12
Tokens: 15,234 in / 3,456 out
API Requests: 7
```

### Token Pricing (Approximate)

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|------------------------|
| Claude Sonnet 4 | $3.00 | $15.00 |
| Claude Opus 4 | $15.00 | $75.00 |
| Claude Haiku 3.5 | $0.80 | $4.00 |

### Cost Optimization Strategies

| Strategy | How |
|----------|-----|
| Right model for task | Haiku for simple reads, Sonnet for coding, Opus for architecture |
| Minimize context | Use `.clineignore`, be specific about files |
| Batch related changes | One prompt for multiple similar edits |
| Start fresh when bloated | New task instead of continuing long conversations |
| Set budget limits | Configure per-task and monthly budgets in settings |

---

## 12. Best Practices & Tips

### Do's ✅

| Practice | Benefit |
|----------|---------|
| Review every file change diff | Catch errors before committing |
| Use `.clinerules` for team consistency | Same AI behavior for everyone |
| Start small, build complexity | Better results, less wasted tokens |
| Commit frequently during sessions | Easy rollback if needed |
| Use git branches for experiments | Safe experimentation |
| Provide file paths when known | Saves context tokens |
| Summarize for follow-up tasks | Better continuity |

### Don'ts ❌

| Anti-Pattern | Risk |
|--------------|------|
| YOLO mode on production code | Unreviewed destructive changes |
| "Rewrite everything" prompts | Massive context, poor results |
| Ignoring the cost counter | Surprise bills |
| No `.clinerules` on team projects | Inconsistent code generation |
| Approving without reading diffs | Subtle bugs, security issues |
| Very long single conversations | Context degradation |
| Sharing API keys in prompts | Security breach |

### Pro Tips

**1. Plan First Pattern:**
```
Before making changes, analyze the codebase and create a plan for 
implementing [feature]. Don't make changes until I approve the plan.
```

**2. Leverage Existing Code:**
```
Look at how UserController.ts handles CRUD and create an identical 
pattern for ProductController.ts.
```

**3. Iterative Refinement:**
```
Good start! Now improve this:
- Add error handling for network timeouts
- Add retry logic with exponential backoff
- Add logging at debug level for each retry
```

**4. Ask Before Acting:**
```
Explain what you think is causing the test failure before attempting a fix.
```

### Workflow for Teams

1. **Standardize `.clinerules`** — Commit to repo
2. **Agree on model selection** — Document which models for which tasks
3. **Share prompt templates** — Create a `prompts/` directory
4. **Same review standards** — AI code gets same PR review as human code
5. **Track costs per team** — Monitor and set budgets

---

## 13. Comparison: Cline vs Claude Code CLI

### Feature Comparison

| Feature | Cline (VS Code) | Claude Code (CLI) |
|---------|-----------------|-------------------|
| **Interface** | VS Code sidebar GUI | Terminal CLI |
| **File editing** | Visual diffs in editor | In-terminal diffs |
| **Approval UX** | Click buttons | Type y/n |
| **MCP support** | ✅ Built-in config UI | ✅ Via config files |
| **Multi-provider** | ✅ Many providers | ❌ Anthropic only |
| **Custom rules** | `.clinerules` file | `CLAUDE.md` file |
| **Cost tracking** | ✅ Real-time UI | ✅ /cost command |
| **Browser tool** | ✅ Built-in | ❌ Via MCP only |
| **Pipe input** | ❌ | ✅ `cat file \| claude` |
| **Headless/CI mode** | ❌ | ✅ `--print` flag |
| **Open source** | ✅ (Apache 2.0) | ❌ (Proprietary) |

### When to Use Which

| Scenario | Better Choice | Why |
|----------|---------------|-----|
| Daily development in VS Code | **Cline** | Integrated into editor |
| Quick one-off commands | **Claude Code** | Faster for simple tasks |
| CI/CD integration | **Claude Code** | Headless mode support |
| Visual file change review | **Cline** | Rich diff viewer |
| Using non-Anthropic models | **Cline** | Multi-provider support |
| Learning/exploring | **Cline** | Better visual feedback |

### Using Both Together

```
Day-to-day workflow:
├── Cline (VS Code)  → Primary development, visual feedback
├── Claude Code (CLI) → Quick fixes, git operations, CI tasks
└── Both share        → Project rules for consistency
```

---

## Key Takeaways

1. **Cline is your AI pair programmer inside VS Code** — reads, writes, executes with your approval
2. **Claude Sonnet 4 is the sweet spot** — Haiku for simple, Opus for complex
3. **`.clinerules` is your contract** — define conventions once, get consistent code
4. **Human-in-the-loop is a feature** — review diffs, approve commands
5. **Context management matters** — `.clineignore`, `@` mentions, fresh tasks
6. **MCP servers extend capabilities** — databases, APIs, search, memory
7. **Cost awareness** — track spending, use the right model for each task
8. **Team standardization** — shared rules + templates = consistent AI output

---

*Last updated: 2025-05-12*