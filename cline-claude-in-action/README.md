# Cline + Claude in Action

A comprehensive guide to using Cline (VS Code AI coding assistant) with Claude models for maximum productivity in software development.

**Inspired by:** The [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action) course structure, adapted for the Cline + Claude workflow.

---

## Table of Contents

1. [Introduction to Cline](#1-introduction-to-cline)
2. [Setting Up Cline with Claude](#2-setting-up-cline-with-claude)
3. [Understanding Cline's Architecture](#3-understanding-clines-architecture)
4. [Core Workflows](#4-core-workflows)
5. [Cline's Tool Use & Capabilities](#5-clines-tool-use--capabilities)
6. [Effective Prompting in Cline](#6-effective-prompting-in-cline)
7. [Custom Instructions & Rules](#7-custom-instructions--rules)
8. [Context Management](#8-context-management)
9. [MCP Servers in Cline](ADVANCED.md#9-mcp-servers-in-cline)
10. [Advanced Workflows](ADVANCED.md#10-advanced-workflows)
11. [Cost Management & Model Selection](ADVANCED.md#11-cost-management--model-selection)
12. [Best Practices & Tips](ADVANCED.md#12-best-practices--tips)
13. [Comparison: Cline vs Claude Code CLI](ADVANCED.md#13-comparison-cline-vs-claude-code-cli)

---

## 1. Introduction to Cline

### What is Cline?

Cline is an open-source AI coding assistant that runs as a VS Code extension. It provides an agentic coding experience where Claude (or other LLMs) can:

- Read and write files in your project
- Execute terminal commands
- Browse the web
- Use MCP (Model Context Protocol) servers
- Interact with your development environment

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Open Source** | Fully open-source (Apache 2.0 license) |
| **IDE Integration** | Runs inside VS Code as an extension |
| **Multi-Provider** | Supports Anthropic, OpenRouter, AWS Bedrock, Google Vertex, OpenAI, and more |
| **Human-in-the-Loop** | Every action requires user approval (configurable) |
| **Transparent** | Shows all API calls, costs, and token usage |
| **Extensible** | Supports MCP servers for custom tool integration |

### Why Cline + Claude?

- **Visual workflow**: See file changes as diffs directly in VS Code
- **Approval-based**: You control what gets executed
- **Context-aware**: Cline intelligently gathers context from your workspace
- **Full IDE integration**: Works with your existing VS Code setup (extensions, settings, terminal)
- **Cost transparency**: Real-time token and cost tracking

---

## 2. Setting Up Cline with Claude

### Installation

1. **Install VS Code** (if not already installed)
   - Download from [code.visualstudio.com](https://code.visualstudio.com/)

2. **Install Cline Extension**
   - Open VS Code
   - Go to Extensions (`Cmd+Shift+X` / `Ctrl+Shift+X`)
   - Search for "Cline"
   - Click Install

3. **Configure API Provider**
   - Open Cline sidebar (robot icon in activity bar)
   - Click the settings gear icon
   - Select "Anthropic" as your API provider
   - Enter your Anthropic API key

### API Key Setup

```
Provider: Anthropic
API Key: sk-ant-api03-xxxxxxxxxxxxx
Model: claude-sonnet-4-20250514 (recommended for coding)
```

### Alternative Providers for Claude

| Provider | Use Case | Setup |
|----------|----------|-------|
| **Anthropic Direct** | Standard usage, full control | API key from console.anthropic.com |
| **OpenRouter** | Access multiple models, pay-per-use | API key from openrouter.ai |
| **AWS Bedrock** | Enterprise, data residency requirements | AWS credentials + region |
| **Google Vertex AI** | GCP-integrated workflows | GCP project + credentials |

### Recommended Model Selection

| Task Type | Recommended Model | Reasoning |
|-----------|-------------------|-----------|
| Complex architecture | Claude Sonnet 4 | Best balance of capability and cost |
| Simple edits/refactoring | Claude Haiku | Fast and cheap |
| Critical/complex decisions | Claude Opus 4 | Highest capability |
| Exploration/prototyping | Claude Sonnet 4 | Good balance |

---

## 3. Understanding Cline's Architecture

### How Cline Works

```
┌─────────────────────────────────────────────┐
│                  VS Code                      │
│  ┌─────────────────────────────────────────┐ │
│  │            Cline Extension               │ │
│  │  ┌───────────┐    ┌──────────────────┐  │ │
│  │  │  Chat UI  │    │  Tool Executor   │  │ │
│  │  └─────┬─────┘    └────────┬─────────┘  │ │
│  │        │                    │             │ │
│  │  ┌─────┴────────────────────┴─────────┐  │ │
│  │  │         Message Handler            │  │ │
│  │  └─────────────────┬─────────────────┘  │ │
│  └────────────────────┼─────────────────────┘ │
│                       │                        │
└───────────────────────┼────────────────────────┘
                        │ API Calls
                        ▼
              ┌──────────────────┐
              │   Claude API     │
              │  (Anthropic)     │
              └──────────────────┘
```

### The Agentic Loop

1. **User sends message** → Cline forwards to Claude with system prompt + context
2. **Claude responds** → May include tool use requests (read file, write file, execute command)
3. **Cline presents action** → Shows user what Claude wants to do
4. **User approves/rejects** → Human-in-the-loop control
5. **Action executes** → Result sent back to Claude
6. **Loop continues** → Until task is complete or user intervenes

### Cline's System Prompt

Cline uses a carefully crafted system prompt that:
- Defines available tools and their schemas
- Provides environment information (OS, shell, working directory)
- Includes custom instructions (if configured)
- Sets behavioral guidelines

---

## 4. Core Workflows

### 4.1 Ask a Question (No File Changes)

Simply ask Claude about your codebase:

```
What design patterns are used in the src/services directory?
```

Cline will:
1. Read relevant files to understand the codebase
2. Provide analysis without making changes

### 4.2 Create New Files

```
Create a new Express.js API endpoint for user authentication 
with JWT tokens. Include input validation and error handling.
```

Cline will:
1. Analyze existing project structure
2. Create new file(s)
3. Show you a diff to approve

### 4.3 Edit Existing Files

```
Refactor the UserService class to use dependency injection 
instead of direct instantiation of dependencies.
```

Cline will:
1. Read the target file(s)
2. Propose changes as a diff
3. Wait for your approval

### 4.4 Execute Commands

```
Run the test suite and fix any failing tests.
```

Cline will:
1. Execute `npm test` (or equivalent)
2. Read test output
3. Identify and fix failures
4. Re-run to verify

### 4.5 Multi-Step Tasks

```
Set up a complete CI/CD pipeline for this project:
1. Create GitHub Actions workflow
2. Add linting configuration
3. Set up test coverage reporting
4. Create a Dockerfile for production
```

Cline will work through each step sequentially, asking for approval at each significant action.

---

## 5. Cline's Tool Use & Capabilities

### Available Tools

| Tool | Description | Example |
|------|-------------|---------|
| `read_file` | Read file contents | Reading source code for context |
| `write_to_file` | Create or overwrite a file | Creating new components |
| `replace_in_file` | Make targeted edits via search/replace | Fixing bugs, refactoring |
| `list_files` | List directory contents | Understanding project structure |
| `list_code_definition_names` | List functions/classes in a file | Quick code overview |
| `search_files` | Regex search across files | Finding usages, patterns |
| `execute_command` | Run terminal commands | Build, test, install deps |
| `browser_action` | Browse web pages | Research, check docs |
| `ask_followup_question` | Ask user for clarification | When task is ambiguous |
| `attempt_completion` | Mark task as complete | Final step |

### Tool Approval Modes

| Mode | Description | Best For |
|------|-------------|----------|
| **Ask Always** | Approve every action | Sensitive codebases, learning |
| **Auto-approve reads** | Auto-approve file reads, ask for writes | Normal development |
| **Auto-approve safe** | Auto-approve reads + non-destructive commands | Trusted workflows |
| **YOLO Mode** | Auto-approve everything | Rapid prototyping (use with caution) |

### Configuring Auto-Approve

In Cline settings, you can configure:
- ✅ Auto-approve read operations
- ✅ Auto-approve list operations
- ⬜ Auto-approve write operations
- ⬜ Auto-approve command execution
- ⬜ Auto-approve browser actions

---

## 6. Effective Prompting in Cline

### Principles for Great Prompts

#### 1. Be Specific About the Outcome

❌ Bad: `Fix the login`

✅ Good:
```
The login form at src/components/LoginForm.tsx is not showing 
validation errors when the email format is invalid. Add client-side 
email validation that shows an inline error message below the email input.
```

#### 2. Provide Context

```
We're using Next.js 14 with the App Router, Tailwind CSS for styling, 
and Zod for validation. Follow the existing patterns in src/components/forms/.
```

#### 3. Specify Constraints

```
Refactor the database queries in src/repositories/UserRepository.ts to:
- Use parameterized queries (prevent SQL injection)
- Add connection pooling
- Keep the same public interface (don't change method signatures)
- Add JSDoc comments for each public method
```

#### 4. Break Complex Tasks into Steps

```
Let's implement the shopping cart feature in phases:
Phase 1 (now): Create the CartContext and CartProvider
Phase 2 (next): Build the CartDrawer UI component
Phase 3 (later): Integrate with the Stripe checkout API
Start with Phase 1.
```

### Prompt Templates

#### Bug Fix Template
```
Bug: [describe the bug]
Expected: [what should happen]
Actual: [what's happening]
Steps to reproduce: [how to trigger it]
Relevant files: [file paths if known]
Please investigate and fix this bug.
```

#### Feature Implementation Template
```
Feature: [feature name]
Requirements:
- [requirement 1]
- [requirement 2]
Technical constraints:
- [constraint 1]
- [constraint 2]
Please implement this following our existing patterns in [reference file/dir].
```

#### Code Review Template
```
Please review the changes in [file path] for:
- Security vulnerabilities
- Performance issues
- Code style consistency
- Missing error handling
- Test coverage gaps
Suggest improvements but don't make changes yet.
```

---

## 7. Custom Instructions & Rules

### What Are Custom Instructions?

Custom instructions are persistent guidelines that Cline includes in every conversation. They shape Claude's behavior for your specific project.

### Setting Up Custom Instructions

**Location:** Cline Settings → Custom Instructions

Or create a `.clinerules` file in your project root (recommended for team sharing):

```
# .clinerules

## Project Context
This is a TypeScript monorepo using:
- Turborepo for build orchestration
- Next.js 14 (App Router) for the web app
- Express.js for the API server
- PostgreSQL with Drizzle ORM
- Vitest for testing

## Coding Standards
- Use functional components with hooks (no class components)
- Prefer named exports over default exports
- Use `type` instead of `interface` for TypeScript types
- Error handling: use Result pattern (never throw in business logic)

## File Naming
- Components: PascalCase (UserProfile.tsx)
- Utilities: camelCase (formatDate.ts)
- Test files: *.test.ts or *.spec.ts

## Testing Requirements
- All new functions must have unit tests
- Use Vitest + Testing Library
- Minimum 80% coverage for new code

## Security
- Never hardcode secrets or API keys
- Always validate and sanitize user input
- Use parameterized queries for all database operations
```

### Rule Files Hierarchy

```
project-root/
├── .clinerules              # Project-wide rules
├── .clineignore             # Files Cline should not read/modify
├── packages/
│   ├── web/
│   │   └── .clinerules     # Web-specific rules
│   └── api/
│       └── .clinerules     # API-specific rules
```

---

## 8. Context Management

### How Context Works in Cline

Cline manages context through:
1. **System prompt** (always included)
2. **Custom instructions** (always included)
3. **Conversation history** (grows over time)
4. **File contents** (added when read)
5. **Command outputs** (added when executed)

### Context Window Limits

| Model | Context Window | Effective for Cline |
|-------|---------------|---------------------|
| Claude Sonnet 4 | 200K tokens | ~150K usable (system prompt overhead) |
| Claude Opus 4 | 200K tokens | ~150K usable |
| Claude Haiku 3.5 | 200K tokens | ~150K usable |

### Strategies for Managing Context

#### 1. Start Fresh When Needed
When context gets large (you'll see token count increasing), click "New Task" to start fresh. Summarize what was accomplished and what's next.

#### 2. Use `@` Mentions for Targeted Context
```
@src/services/UserService.ts refactor the createUser method 
to validate email uniqueness before insertion
```

#### 3. Keep Tasks Focused
Break work into smaller conversations rather than one massive task.

#### 4. Use .clineignore to Reduce Noise
```
# .clineignore
node_modules/
dist/
build/
.git/
*.lock
coverage/
```

#### 5. Provide Summaries for New Conversations
```
Context: We just finished implementing the UserService with CRUD 
operations and JWT authentication. Now I need to add email verification.
```

---

**Continue to [Advanced Topics (Sections 9-13) →](ADVANCED.md)**

---

*Last updated: 2025-05-12*