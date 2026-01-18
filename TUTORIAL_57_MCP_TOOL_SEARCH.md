# Tutorial 57: MCP Tool Search & Context Optimization

> **Managing MCP tools efficiently with auto-deferral and context-aware loading**

## Table of Contents

1. [Overview](#overview)
2. [The Context Window Problem](#the-context-window-problem)
3. [MCP Tool Search Auto Mode](#mcp-tool-search-auto-mode)
4. [Configuration](#configuration)
5. [Tool Deferral Strategies](#tool-deferral-strategies)
6. [Monitoring Context Usage](#monitoring-context-usage)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)
9. [Quick Reference](#quick-reference)

---

## Overview

As of Claude Code v2.1.7 and v2.1.9, MCP Tool Search introduces intelligent context management for projects with many MCP tools. This feature automatically defers tool definitions that would consume too much of the context window.

### The Problem

MCP servers can expose dozens or hundreds of tools. Each tool definition consumes context tokens:

```
Example MCP Server: 50 tools × ~200 tokens/tool = 10,000 tokens
Multiple Servers: 200 tools × ~200 tokens/tool = 40,000 tokens
```

With a 200K context window, 40K tokens for tool definitions leaves less room for:
- File contents
- Conversation history
- Claude's reasoning
- Output generation

### The Solution

MCP Tool Search uses auto-deferral to:
1. Monitor context usage from tool definitions
2. Defer tool definitions that exceed a threshold
3. Load deferred tools on-demand when needed
4. Keep essential tools always available

---

## The Context Window Problem

### Understanding Token Budgets

```
┌─────────────────────────────────────────────────────────────────┐
│                    Context Window (200K tokens)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐                                            │
│  │ System Prompt   │ ~5K tokens                                 │
│  └─────────────────┘                                            │
│  ┌─────────────────┐                                            │
│  │ Tool Definitions│ Variable (can be 40K+ with many MCP tools) │
│  └─────────────────┘                                            │
│  ┌─────────────────┐                                            │
│  │ Conversation    │ Variable                                    │
│  └─────────────────┘                                            │
│  ┌─────────────────┐                                            │
│  │ File Contents   │ Variable                                    │
│  └─────────────────┘                                            │
│  ┌─────────────────┐                                            │
│  │ Output Space    │ ~16K tokens reserved                        │
│  └─────────────────┘                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Symptoms of Context Pressure

- Slower response times
- Truncated file reads
- Early conversation summarization
- Missing context from earlier messages
- "Context limit reached" warnings

### Why Tool Definitions Are Expensive

Each tool definition includes:
- Name and description
- Parameter schema (JSON Schema)
- Examples and documentation
- Return type information

Example tool definition size:

```json
{
  "name": "mcp__database__query",
  "description": "Execute SQL query against database...",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "SQL query to execute"
      },
      "database": {
        "type": "string",
        "enum": ["primary", "replica", "analytics"]
      },
      "timeout": {
        "type": "number",
        "description": "Query timeout in seconds"
      }
    },
    "required": ["query"]
  }
}
// ~300 tokens for this single tool
```

---

## MCP Tool Search Auto Mode

### How Auto Mode Works

Auto mode (`auto:N`) monitors the percentage of context used by tool definitions:

1. **Threshold Check**: If tool definitions exceed N% of context
2. **Deferral**: Non-essential tools are deferred (not loaded upfront)
3. **On-Demand Loading**: Deferred tools are searched and loaded when needed
4. **Smart Selection**: Claude describes what tool it needs, system finds it

### Configuration Syntax

```json
{
  "mcpToolSearch": "auto:10"
}
```

This means: "Defer tools when they exceed 10% of the context window."

### Threshold Options

| Setting | Behavior | Use Case |
|---------|----------|----------|
| `auto:5` | Aggressive deferral (>5% triggers) | Very large tool sets |
| `auto:10` | Balanced (default recommendation) | Most projects |
| `auto:15` | Conservative deferral | Smaller tool sets |
| `auto:20` | Minimal deferral | Few MCP servers |
| `disabled` | No deferral | Debug/testing |

### Environment Variable Alternative

You can also control tool search via the `ENABLE_TOOL_SEARCH` environment variable:

```bash
# Command line
ENABLE_TOOL_SEARCH=auto:5 claude

# Or export before starting
export ENABLE_TOOL_SEARCH=auto:10
claude
```

| Value | Behavior |
|-------|----------|
| `auto` | Activates at >10% context (default) |
| `auto:<N>` | Custom threshold (e.g., `auto:5` for 5%) |
| `true` | Always enabled |
| `false` | Disabled, all tools loaded upfront |

**Via settings.json env field:**

```json
{
  "env": {
    "ENABLE_TOOL_SEARCH": "auto:5"
  }
}
```

> **Note:** The `mcpToolSearch` setting and `ENABLE_TOOL_SEARCH` env var control the same feature. Use whichever fits your workflow.

### Example Scenarios

**Scenario 1: Small Project (20 tools)**
```
20 tools × 200 tokens = 4,000 tokens
Context: 200,000 tokens
Tool %: 2%

With auto:10 → No deferral needed (under threshold)
```

**Scenario 2: Large Project (200 tools)**
```
200 tools × 200 tokens = 40,000 tokens
Context: 200,000 tokens
Tool %: 20%

With auto:10 → Deferral triggered
~150 tools deferred → ~10,000 tokens for tools
```

---

## Configuration

### Global Settings

In `~/.claude/settings.json`:

```json
{
  "mcpToolSearch": "auto:10"
}
```

### Project Settings

In `.claude/settings.json`:

```json
{
  "mcpToolSearch": "auto:15"
}
```

### Per-Session Override

```bash
# Not currently available via CLI
# Configure in settings file
```

### Complete Configuration Example

```json
{
  "mcpToolSearch": "auto:10",
  "mcp": {
    "servers": {
      "database": {
        "command": "mcp-server-database",
        "priority": "high"  // Won't be deferred
      },
      "analytics": {
        "command": "mcp-server-analytics",
        "priority": "low"   // Can be deferred
      }
    }
  }
}
```

### Disabling Auto Mode

```json
{
  "mcpToolSearch": "disabled"
}
```

---

## Tool Deferral Strategies

### Which Tools Get Deferred?

The system uses several factors to decide deferral order:

1. **Usage Frequency**: Rarely-used tools deferred first
2. **Tool Priority**: Low-priority MCP servers deferred first
3. **Tool Size**: Larger definitions deferred first
4. **Recency**: Recently-used tools kept

### Deferral Priority Order

```
NEVER DEFERRED (Core Tools):
├── Read, Write, Edit
├── Glob, Grep
├── Bash (if allowed)
├── Task, TodoWrite
└── WebFetch, WebSearch

DEFERRED LAST (High Priority MCP):
├── Explicitly marked high-priority servers
└── Recently-used MCP tools

DEFERRED FIRST (Low Priority):
├── Explicitly marked low-priority servers
├── Large tool definitions
└── Rarely-used tools
```

### Marking Server Priority

```json
{
  "mcp": {
    "servers": {
      "critical-api": {
        "command": "mcp-server-api",
        "priority": "high"
      },
      "documentation": {
        "command": "mcp-server-docs",
        "priority": "low"
      }
    }
  }
}
```

### Tool Groups

Group related tools to control deferral together:

```json
{
  "mcp": {
    "servers": {
      "database": {
        "command": "mcp-server-db",
        "toolGroups": {
          "read": ["query", "list_tables", "describe"],
          "write": ["insert", "update", "delete"]
        },
        "deferGroups": ["write"]  // Defer write operations
      }
    }
  }
}
```

---

## Monitoring Context Usage

### Checking Current Usage

```
/cost
```

Shows token usage breakdown including tool definitions.

### Stats Command

```
/stats

Session Statistics:
- Total tokens: 45,230
- Tool definitions: 8,450 (18.7%)
- Conversation: 28,000
- Files read: 8,780
- MCP tools available: 180
- MCP tools deferred: 145
```

### Verbose Mode

For debugging, use verbose mode:

```bash
claude --verbose
```

This shows:
- Which tools are loaded vs deferred
- Deferral decisions being made
- On-demand tool loading events

### Context Pressure Indicators

Watch for these signs of context pressure:

| Indicator | Meaning | Action |
|-----------|---------|--------|
| Frequent summarization | Context filling up | Lower threshold |
| "Searching for tool" messages | Tools being loaded on-demand | Working correctly |
| Slow tool responses | Many deferred tools | Check MCP server health |
| Missing earlier context | Window exhausted | Lower threshold, /compact |

---

## Best Practices

### 1. Start with Reasonable Threshold

```json
{
  "mcpToolSearch": "auto:10"
}
```

10% is a good starting point for most projects.

### 2. Mark Critical Tools High Priority

```json
{
  "mcp": {
    "servers": {
      "your-main-api": {
        "command": "...",
        "priority": "high"
      }
    }
  }
}
```

### 3. Use Selective MCP Server Loading

Only load MCP servers you need for the current task:

```bash
claude --mcp-config ./minimal-mcp.json
```

Create task-specific MCP configurations:

```json
// mcp-config-dev.json - Development tools only
{
  "mcpServers": {
    "database": { "command": "mcp-db" },
    "git": { "command": "mcp-git" }
  }
}

// mcp-config-deploy.json - Deployment tools only
{
  "mcpServers": {
    "k8s": { "command": "mcp-kubernetes" },
    "aws": { "command": "mcp-aws" }
  }
}
```

### 4. Monitor and Adjust

Periodically check context usage:

```
/stats
/cost
```

If tools consistently consume >15% of context, consider:
- Lowering the threshold
- Splitting into multiple MCP configs
- Removing unused MCP servers

### 5. Use Tool Groups Strategically

If an MCP server has read and write tools, consider deferring write tools:

```json
{
  "mcp": {
    "servers": {
      "database": {
        "deferTools": ["insert", "update", "delete", "truncate"]
      }
    }
  }
}
```

### 6. Handle Deferred Tools Gracefully

When Claude needs a deferred tool:

```
Claude: I need to query the analytics database. Let me search
for the appropriate tool...

[Searching deferred tools for "analytics query"]
[Loading mcp__analytics__run_query]

Now executing the query...
```

This is normal behavior and indicates the system is working.

---

## Troubleshooting

### Tools Not Being Deferred

**Symptom**: Tool definitions still consuming too much context

**Check**:
1. Verify `mcpToolSearch` is configured
2. Check if tools are marked high priority
3. Verify threshold isn't too high

**Solution**:
```json
{
  "mcpToolSearch": "auto:5"  // More aggressive deferral
}
```

### Too Aggressive Deferral

**Symptom**: Claude frequently searching for tools, slow responses

**Check**:
1. Threshold may be too low
2. Many tools marked low priority

**Solution**:
```json
{
  "mcpToolSearch": "auto:15"  // Less aggressive
}
```

Or mark important tools high priority:
```json
{
  "mcp": {
    "servers": {
      "main-api": {
        "priority": "high"
      }
    }
  }
}
```

### Deferred Tool Not Found

**Symptom**: "Cannot find tool matching..." errors

**Check**:
1. MCP server is running
2. Tool exists in server
3. Tool name is correct

**Debug**:
```bash
# List all tools from MCP server
claude mcp list
```

### Performance Issues

**Symptom**: Slow tool loading, timeouts

**Check**:
1. MCP server health
2. Network connectivity
3. Server startup time

**Solution**:
- Increase MCP timeout in settings
- Pre-warm MCP servers
- Use local MCP servers when possible

---

## Quick Reference

### Configuration Options

```json
{
  "mcpToolSearch": "auto:10",      // 10% threshold
  "mcpToolSearch": "auto:5",       // 5% (aggressive)
  "mcpToolSearch": "auto:20",      // 20% (conservative)
  "mcpToolSearch": "disabled"      // No deferral
}
```

### Server Priority

```json
{
  "mcp": {
    "servers": {
      "server-name": {
        "priority": "high",   // Never deferred
        "priority": "normal", // Default
        "priority": "low"     // Deferred first
      }
    }
  }
}
```

### Monitoring Commands

```
/cost         # Token usage breakdown
/stats        # Session statistics
/config mcp   # MCP configuration
```

### Threshold Guidelines

| Tool Count | Recommended Threshold |
|------------|----------------------|
| < 50 | `auto:20` or `disabled` |
| 50-100 | `auto:15` |
| 100-200 | `auto:10` |
| > 200 | `auto:5` |

### Context Budget Guidelines

| Component | Recommended % |
|-----------|---------------|
| System prompt | 2-3% |
| Tool definitions | 5-15% |
| Conversation | 40-60% |
| File contents | 20-40% |
| Output reserve | 8% |

---

## Advanced: Custom Tool Deferral

### Explicit Tool Deferral

Force specific tools to always be deferred:

```json
{
  "mcp": {
    "servers": {
      "database": {
        "alwaysDefer": [
          "admin_drop_table",
          "admin_truncate",
          "admin_create_index"
        ]
      }
    }
  }
}
```

### Never Defer List

Ensure specific tools are never deferred:

```json
{
  "mcp": {
    "servers": {
      "api": {
        "neverDefer": [
          "list_endpoints",
          "health_check",
          "get_status"
        ]
      }
    }
  }
}
```

### Dynamic Deferral Based on Task

Create task-specific MCP configs and switch between them:

```bash
# Development mode - full tool access
claude --mcp-config ./mcp-dev.json

# Production mode - read-only tools
claude --mcp-config ./mcp-readonly.json

# Specific task - minimal tools
claude --mcp-config ./mcp-minimal.json
```

---

## Sources

- [MCP Tool Search](https://code.claude.com/docs/en/mcp.md)
- [Settings Reference](https://code.claude.com/docs/en/settings.md)
- [Claude Code Changelog v2.1.7, v2.1.9](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
