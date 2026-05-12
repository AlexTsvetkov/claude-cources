# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a documentation repository containing course summaries from Anthropic's Skilljar training platform. The content is organized into course-specific directories, each containing a shortened reference guide for developers, architects, and technical leads.

**Purpose:** Quick-reference summaries for enterprise Java developers adopting Claude and AI tools in their workflows.

## Repository Structure

```
claude-cources/
├── Claude-101/                           # Intro to Claude fundamentals
├── ai-fluency-framework-foundations/     # Building AI literacy framework
├── cline-claude-in-action/               # Using Cline + Claude in VS Code
├── claude-code-in-action/                # Using Claude Code CLI tool
├── claude-with-the-anthropic-api/        # API integration guide
└── introduction-to-model-context-protocol/ # MCP protocol overview
```

Each directory contains a `README.md` file formatted as a course summary with:
- Table of contents
- Structured sections (numbered headings)
- Code examples (Java, Python, TypeScript)
- Best practices tables
- Architecture diagrams (ASCII art)
- Quick reference sections

## Course Topics

| Course | Audience | Key Topics |
|--------|----------|-----------|
| **Claude-101** | Beginners | Core capabilities, prompt engineering, CRAFT framework |
| **AI Fluency Framework** | All devs | Mental models, evaluation framework (VALID), workflow integration |
| **Claude with the Anthropic API** | Backend devs | Messages API, tool use, streaming, vision, error handling |
| **Cline + Claude in Action** | VS Code users | Cline setup, agentic workflows, .clinerules, MCP, cost management |
| **Claude Code in Action** | CLI users | Agentic execution, CLAUDE.md usage, multi-file workflows |
| **Model Context Protocol** | Architects | MCP architecture, building servers, security considerations |

## Documentation Conventions

### File Format
- All course summaries are in Markdown
- Title format: `# [Course Name] — Shortened Course Summary`
- Source citation at the top with Skilljar link
- Audience and purpose clearly stated
- Last updated date at the bottom

### Content Structure
- Use numbered sections (##) for main topics
- Use tables for comparisons, parameters, and reference data
- Code blocks with language hints (\`\`\`java, \`\`\`json, etc.)
- Consistent use of bold (**) for emphasis
- ASCII diagrams for architecture visualization

### Style Guidelines
- Target audience: Senior Java developers / architects in enterprise contexts
- Tone: Professional, concise, technical
- Examples: Prefer Java/Spring Boot where language-agnostic isn't required
- Focus: Practical application over theory

## Working with This Repository

### Adding New Course Summaries
When adding a new course summary:
1. Create a new directory with lowercase-with-hyphens naming
2. Add a README.md following the existing structure
3. Include source URL, audience, and purpose in the header
4. Use consistent section numbering and formatting
5. Add a "Key Takeaways" section at the end
6. Include last-updated date

### Updating Existing Content
- Preserve the existing structure and formatting
- Update the "Last updated" date at the bottom
- Maintain consistency with sibling course summaries
- Keep code examples current with latest API versions

### Cross-Referencing
When content overlaps between courses:
- Reference other summaries in the same repo
- Use relative links: `[Claude-101](../Claude-101/README.md)`
- Avoid duplicating detailed content; link instead

## Git Workflow

### Commits
- Use conventional commit format: `type: description`
- Types: `Add`, `Update`, `Fix`, `Refactor`, `Remove`
- Examples:
  - `Add: new course summary for Advanced Prompt Engineering`
  - `Update: Claude API course with Sonnet 4.6 references`
  - `Fix: typo in MCP architecture diagram`

### Content Updates
Since this is a documentation repo with no CI/CD or build process:
- Direct commits to `main` are acceptable
- No tests to run
- Changes are immediately visible

## Notes

- This repository contains no executable code or build artifacts
- All content is read-only documentation/reference material
- The parent directory contains a generic CLAUDE.md template; this repo-specific file takes precedence
- Jira credentials are stored in the parent directory's CLAUDE.md (not this repo)