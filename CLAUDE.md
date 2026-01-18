# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive Claude Code tutorial collection with in-depth educational markdown files. The project teaches Claude Code mastery from beginner to advanced enterprise usage.

**Type**: Documentation/Educational
**License**: MIT
**Primary Content**: Markdown tutorials

## Repository Structure

```
├── README.md                      # Main index with 5 learning paths
├── AGENT_WORKFLOWS_TUTORIAL.md    # 7-phase development lifecycle guide
├── TROUBLESHOOTING_GIT_PUSH.md    # Git secret scanning debug guide
├── TUTORIAL_01-10                 # Core features (hooks, commands, subagents, MCP, skills, bash, practices, workflows, settings, IDE)
├── TUTORIAL_11-25                 # Advanced topics (worktrees, teleport, CI/CD, plugins, headless, SDK)
├── TUTORIAL_26-35                 # Workflow topics (background tasks, checkpointing, cost optimization)
├── TUTORIAL_36-42                 # Security & enterprise (permissions, sandbox, Bedrock, Vertex AI)
└── TUTORIAL_43-53                 # Practical workflows (debugging, testing, PR, code review)
```

## Learning Paths

1. **Quick Start** - Terminal basics, slash commands, best practices
2. **Power User** - Subagents, worktrees, background tasks, skills, hooks
3. **Enterprise** - Settings, permissions, sandbox, Bedrock, Vertex AI
4. **CI/CD** - GitHub Actions, GitLab, headless mode, Agent SDK
5. **IDE Mastery** - VS Code, JetBrains, keyboard shortcuts

## Key Tutorials by Topic

| Topic | Tutorial |
|-------|----------|
| CLAUDE.md writing | TUTORIAL_21_CLAUDE_MD_MASTERY.md |
| Best practices | TUTORIAL_07_BEST_PRACTICES.md |
| Development phases | AGENT_WORKFLOWS_TUTORIAL.md |
| Terminal/Bash | TUTORIAL_06_TERMINAL_BASH.md (largest, 156KB) |
| IDE setup | TUTORIAL_10_IDE_INTEGRATIONS.md |

## Content Conventions

### Tutorial Structure
Each tutorial follows a consistent format:
- Introduction with prerequisites
- Step-by-step guides with examples
- Real-world use cases
- Quick reference/cheat sheet section
- Troubleshooting tips

### Code Examples
- Language-tagged code blocks with explanations
- Both correct and incorrect patterns shown
- JSON for configuration examples
- Shell commands for CLI operations

## When Editing Tutorials

- Maintain the existing structure and formatting style
- Include prerequisites at the start
- Provide real-world examples, not just theory
- Add to quick reference sections when introducing new concepts
- Cross-reference related tutorials where relevant
- Test any code examples for accuracy

## Official Claude Code Documentation

Reference the official docs at https://code.claude.com/docs when updating tutorials.

### Documentation Structure

| Section | Topics |
|---------|--------|
| Getting Started | overview, quickstart, common-workflows |
| IDE Integration | VS Code, JetBrains, Chrome (beta), Desktop, Slack |
| Build with Claude Code | sub-agents, plugins, skills, hooks, MCP, headless |
| Deployment | Amazon Bedrock, Google Vertex AI, Microsoft Foundry, sandboxing |
| Administration | IAM, security, monitoring-usage, costs, analytics |
| Configuration | settings, memory, model-config, terminal-config, statusline |
| Reference | cli-reference, slash-commands, hooks, plugins-reference |

### Key Official Doc URLs

- Documentation map: `https://code.claude.com/docs/en/claude_code_docs_map.md`
- LLM-friendly index: `https://code.claude.com/docs/llms.txt`
- Changelog: `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md`

### Recent Claude Code Features (v2.1.x)

Keep tutorials updated with these recent additions:
- **LSP tool** - go-to-definition, find references, hover docs
- **Chrome integration** - browser control from terminal
- **Skill hot-reload** - automatic updates without restart
- **Sub-agent context forking** - `context: fork` in skill frontmatter
- **MCP tool search auto-mode** - defers tools >10% of context window
- **Wildcard permissions** - `Bash(npm *)`, `Bash(git * main)`
- **Extended Vim motions** - `;`, `,`, `y`, `p`, text objects, `>>`, `<<`

### Configuration Hierarchy

From highest to lowest priority:
1. Command-line arguments (`--model`, `--permission-mode`)
2. Environment variables (`ANTHROPIC_API_KEY`, `CLAUDE_CODE_*`)
3. Workspace settings (`.claude/settings.local.json`)
4. Project settings (`.claude/settings.json`)
5. User settings (`~/.claude/settings.json`)
6. Default settings (built-in)

### Key Environment Variables

| Variable | Purpose |
|----------|---------|
| `CLAUDE_CODE_SHELL` | Override shell detection |
| `CLAUDE_CODE_TMPDIR` | Override temp directory |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | File read token limits |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Disable backgrounding |
