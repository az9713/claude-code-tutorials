# Tutorial 52: Environment Variables

> **Configure Claude Code with environment variables**

## Overview

Environment variables for configuring Claude Code behavior.

---

## Part 1: Core Variables

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API authentication |
| `ANTHROPIC_MODEL` | Default model |
| `CLAUDE_CODE_USE_BEDROCK` | Use AWS Bedrock |
| `CLAUDE_CODE_USE_VERTEX` | Use GCP Vertex |

---

## Part 2: Model Configuration

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Opus alias model |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Sonnet alias model |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Haiku alias model |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Subagent model |

---

## Part 3: Performance

| Variable | Purpose |
|----------|---------|
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Compaction threshold |
| `DISABLE_PROMPT_CACHING` | Disable caching |

---

## Part 4: Cloud Providers

### AWS Bedrock

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-west-2
```

### Google Vertex

```bash
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=us-east5
export ANTHROPIC_VERTEX_PROJECT_ID=my-project
```

---

## Part 5: Setting Variables

### Shell Profile

```bash
# ~/.bashrc or ~/.zshrc
export ANTHROPIC_API_KEY=YOUR_KEY
export ANTHROPIC_MODEL=sonnet
```

### Per Session

```bash
ANTHROPIC_MODEL=opus claude
```

### In Scripts

```bash
#!/bin/bash
export ANTHROPIC_MODEL=haiku
claude -p "Quick task"
```

---

## Quick Reference

| Category | Variables |
|----------|-----------|
| Auth | `ANTHROPIC_API_KEY` |
| Model | `ANTHROPIC_MODEL`, `*_DEFAULT_*_MODEL` |
| Cloud | `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX` |
| Performance | `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` |

---

## Sources

- [Settings](https://code.claude.com/docs/en/settings.md)
- [Model Configuration](https://code.claude.com/docs/en/model-config.md)
