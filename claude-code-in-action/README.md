# Claude Code in Action — Shortened Course Summary

> Source: [Anthropic Skilljar — Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)
> Audience: Developers, Architects, Technical Leads
> Purpose: Practical guide to using Claude Code — Anthropic's agentic CLI coding tool

---

## Table of Contents

1. [What is Claude Code?](#1-what-is-claude-code)
2. [Installation & Setup](#2-installation--setup)
3. [Core Concepts](#3-core-concepts)
4. [Basic Usage](#4-basic-usage)
5. [Working with Codebases](#5-working-with-codebases)
6. [Advanced Workflows](#6-advanced-workflows)
7. [Configuration & Customization](#7-configuration--customization)
8. [Best Practices](#8-best-practices)
9. [Integration with Dev Tools](#9-integration-with-dev-tools)
10. [Key Takeaways](#10-key-takeaways)

---

## 1. What is Claude Code?

Claude Code is an **agentic CLI tool** that lets Claude directly interact with your codebase. Unlike chat-based interactions, Claude Code can:

- Read and write files
- Execute shell commands
- Navigate your project structure
- Run tests and interpret results
- Make multi-file changes autonomously
- Use git for version control

### Claude Code vs. Chat vs. API

| Feature | Claude Chat | Anthropic API | Claude Code |
|---|---|---|---|
| Reads your files | No (paste manually) | No (send in context) | Yes (direct access) |
| Writes files | No | No | Yes |
| Runs commands | No | No (unless tool use) | Yes |
| Multi-file edits | No | No | Yes |
| Git integration | No | No | Yes |
| Agentic loops | No | Manual orchestration | Built-in |

---

## 2. Installation & Setup

### Install

```bash
# Install globally via npm
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version
```

### Authentication

```bash
# Option 1: API key
export ANTHROPIC_API_KEY="sk-ant-api03-..."

# Option 2: Interactive login
claude login
```

### First Run

```bash
# Navigate to your project
cd /path/to/your/spring-boot-project

# Start Claude Code
claude
```

Claude Code automatically detects your project type, language, build tools, and structure.

---

## 3. Core Concepts

### Agentic Execution Model

```
You give a task
    -> Claude reads relevant files
    -> Claude plans approach
    -> Claude makes changes (files, commands)
    -> Claude verifies (runs tests, checks compilation)
    -> Claude reports results
    -> You approve/reject/refine
```

### Permission Model

Claude Code asks for permission before:
- Writing/modifying files
- Executing shell commands
- Installing dependencies

You can configure auto-approval for trusted operations.

### Context Awareness

Claude Code automatically understands:
- Project structure (Maven/Gradle, package layout)
- Dependencies (pom.xml / build.gradle)
- Existing patterns (naming conventions, architecture style)
- Git history (recent changes, branches)

---

## 4. Basic Usage

### Interactive Mode

```bash
claude
```

Then type natural language requests:

```
> Add input validation to the UserController POST endpoint

> Write unit tests for OrderService.calculateTotal()

> Explain what the PaymentGateway class does

> Fix the NullPointerException in CustomerRepository.findByEmail()
```

### One-shot Mode

```bash
# Single command, no interactive session
claude "Add @NotNull annotations to all DTO fields"

# Pipe input
cat error.log | claude "Analyze this stack trace and fix the root cause"
```

### Common Commands

| Command | Purpose |
|---|---|
| claude | Start interactive session |
| claude "task" | One-shot execution |
| claude --resume | Resume previous session |
| claude --print | Output only, no file changes |
| claude config | View/edit configuration |

---

## 5. Working with Codebases

### Project Understanding

When you start Claude Code in a project directory, it:
1. Scans project structure
2. Reads build files (pom.xml, build.gradle)
3. Identifies frameworks (Spring Boot, Quarkus, etc.)
4. Understands test structure
5. Detects code style patterns

### Effective Requests for Java Projects

#### Code Generation
```
> Create a REST endpoint for CRUD operations on Product entity.
  Use Spring Data JPA, return ResponseEntity, include pagination.
  Follow the same patterns as the existing OrderController.
```

#### Refactoring
```
> Refactor CustomerService to use the Strategy pattern for
  discount calculation. Keep backward compatibility.
```

#### Bug Fixing
```
> The integration test CustomerControllerIT.testCreateCustomer is failing
  with a 500 error. Investigate and fix it.
```

#### Test Generation
```
> Write comprehensive unit tests for PaymentService.
  Cover happy path, insufficient funds, expired card, and network timeout.
  Use JUnit 5 + Mockito. Follow AAA pattern.
```

#### Documentation
```
> Generate Javadoc for all public methods in the service layer.
  Include @param, @return, @throws annotations.
```

---

## 6. Advanced Workflows

### Multi-step Tasks

Claude Code excels at complex, multi-file operations:

```
> Implement a caching layer for the ProductService:
  1. Add Spring Cache dependency
  2. Configure Redis as cache provider
  3. Add @Cacheable annotations to read methods
  4. Add @CacheEvict to write methods
  5. Write integration tests with embedded Redis
  6. Update application.yml for all profiles
```

### Git Workflows

```
> Create a feature branch, implement the user notification feature,
  commit with conventional commit messages, and show me the diff.

> Review the changes in the last 3 commits and identify potential issues.

> Resolve the merge conflicts in the current branch.
```

### Test-Driven Development

```
> Using TDD approach:
  1. Write failing tests for a new EmailValidator utility
  2. Implement the minimum code to pass
  3. Refactor for clean code
  Show me each step.
```

### Migration Tasks

```
> Migrate the DAO layer from raw JDBC to Spring Data JPA:
  - Convert SQL queries to JPA repository methods
  - Create entity classes from existing table schema
  - Update service layer to use new repositories
  - Ensure all existing tests still pass
```

### Code Review

```
> Review src/main/java/com/example/service/ for:
  - Thread safety issues
  - Missing error handling
  - SOLID violations
  - Performance concerns
  Format as a review table with severity levels.
```

---

## 7. Configuration & Customization

### CLAUDE.md File

Create a CLAUDE.md in your project root to give Claude persistent context:

```markdown
# Project: Order Management System

## Architecture
- Spring Boot 3.2 + Java 21
- Hexagonal architecture (ports & adapters)
- PostgreSQL + Redis

## Conventions
- Package structure: com.company.module.{domain,application,infrastructure,api}
- DTOs suffixed with Request/Response
- All endpoints return ResponseEntity
- Use MapStruct for mapping
- Tests: JUnit 5 + Mockito + Testcontainers

## Important Rules
- Never use field injection, always constructor injection
- All public API methods must have Javadoc
- Integration tests use @Testcontainers
- No business logic in controllers
```

### Configuration File (.claude/config.json)

```json
{
  "permissions": {
    "allow_file_write": true,
    "allow_commands": ["mvn", "gradle", "git", "docker"],
    "deny_commands": ["rm -rf", "sudo"],
    "auto_approve_patterns": ["src/test/**"]
  },
  "preferences": {
    "language": "java",
    "test_framework": "junit5",
    "style": "google-java-format"
  }
}
```

---

## 8. Best Practices

### Do's

| Practice | Why |
|---|---|
| Start with small, focused tasks | Easier to verify, less risk |
| Use CLAUDE.md for project context | Consistent output, follows your patterns |
| Review changes before accepting | Claude can make mistakes |
| Run tests after changes | Catch regressions immediately |
| Use git branches for experiments | Easy rollback |
| Be specific about constraints | "Use Java 21 features" vs "update the code" |

### Don'ts

| Anti-Pattern | Risk |
|---|---|
| "Refactor everything" | Massive unreviewed changes |
| Accepting without reading | Subtle bugs slip through |
| Skipping tests | No safety net |
| No CLAUDE.md in team projects | Inconsistent AI-generated code |
| Sharing secrets in prompts | Security risk |
| Running on production code without backup | Obvious risk |

### Review Checklist for AI-Generated Code

- [ ] Does it compile? (mvn compile)
- [ ] Do existing tests pass? (mvn test)
- [ ] Does the new code have tests?
- [ ] Is error handling complete?
- [ ] Are edge cases covered?
- [ ] Does it follow project conventions?
- [ ] No hardcoded values or TODO items?
- [ ] No security issues? (SQL injection, XSS, etc.)

---

## 9. Integration with Dev Tools

### IDE Integration

Claude Code works alongside your IDE:
- Make changes via CLI -> see updates in IntelliJ
- Use IntelliJ for navigation, Claude Code for generation
- Run/debug from IDE, fix via Claude Code

### CI/CD Integration

```bash
# In CI pipeline: use Claude for automated code review
claude --print "Review the diff for this PR and list issues" < pr_diff.patch

# Generate release notes
claude --print "Generate release notes from git log since last tag"
```

### Git Hooks

```bash
# pre-commit hook: auto-generate missing Javadoc
claude "Add missing Javadoc to any new public methods in staged files"
```

### Slash Commands

| Command | Purpose |
|---|---|
| /help | Show available commands |
| /clear | Clear conversation context |
| /compact | Summarize and compress context |
| /config | Show current configuration |
| /cost | Show token usage and cost |
| /diff | Show pending file changes |

---

## 10. Key Takeaways

### TL;DR for the Java Architect

1. **Claude Code is agentic** — it reads, writes, executes, and verifies autonomously
2. **CLAUDE.md is your contract** — define conventions once, enforce everywhere
3. **Start small** — single-file tasks first, then multi-file workflows
4. **Always verify** — compilation, tests, code review before accepting
5. **Git branches for safety** — experiment without risk
6. **Complements your IDE** — use both; each has strengths
7. **Team standardization** — shared CLAUDE.md + config = consistent AI output

### Quick-Start for Java Projects

```bash
# 1. Install
npm install -g @anthropic-ai/claude-code

# 2. Navigate to project
cd your-spring-boot-project

# 3. Create CLAUDE.md with your conventions

# 4. Start
claude

# 5. Try a simple task
> Write unit tests for UserService.createUser()
```

### When to Use Claude Code vs. Other Tools

| Scenario | Best Tool |
|---|---|
| Quick question about code | Claude Chat |
| Programmatic AI integration | Anthropic API |
| Multi-file code changes | Claude Code |
| Code generation in IDE | IDE AI plugin |
| Automated code review in CI | Claude Code (--print mode) |

---

*Last updated: 2025-05-01*
