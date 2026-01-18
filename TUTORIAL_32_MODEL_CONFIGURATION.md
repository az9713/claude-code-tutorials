# Tutorial 32: Model Configuration

> **Configure and optimize Claude model selection**

## Overview

Claude Code supports multiple models. Understanding model configuration helps optimize for speed, quality, and cost.

### What You'll Learn

- Available models and aliases
- Setting your preferred model
- Special modes like opusplan
- Extended context windows

---

## Part 1: Available Models

### Model Aliases

| Alias | Description |
|-------|-------------|
| `default` | Recommended based on account |
| `sonnet` | Claude Sonnet 4.5 - Daily coding |
| `opus` | Claude Opus 4.5 - Complex reasoning |
| `haiku` | Claude Haiku - Simple, fast tasks |
| `sonnet[1m]` | Sonnet with 1M context |
| `opusplan` | Opus for planning, Sonnet for execution |

---

## Part 2: Setting Your Model

### Priority Order

1. **During session** - `/model <alias>`
2. **At startup** - `claude --model <alias>`
3. **Environment variable** - `ANTHROPIC_MODEL`
4. **Settings file** - `model` field

### Examples

```bash
# During session
/model opus

# At startup
claude --model opus

# Environment
export ANTHROPIC_MODEL=opus

# Settings file (~/.claude/settings.json)
{
  "model": "opus"
}
```

---

## Part 3: Special Modes

### opusplan Mode

Hybrid approach:
- **Planning mode**: Uses Opus for complex reasoning
- **Execution mode**: Switches to Sonnet for code generation

```bash
/model opusplan
```

### Extended Context [1m]

For long sessions:

```bash
/model sonnet[1m]
```

Enables 1 million token context window.

---

## Part 4: Environment Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_MODEL` | Set model directly |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Override opus alias |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Override sonnet alias |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Override haiku alias |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Model for subagents |

### Prompt Caching

| Variable | Description |
|----------|-------------|
| `DISABLE_PROMPT_CACHING` | Disable all caching |
| `DISABLE_PROMPT_CACHING_HAIKU` | Disable for Haiku |
| `DISABLE_PROMPT_CACHING_SONNET` | Disable for Sonnet |
| `DISABLE_PROMPT_CACHING_OPUS` | Disable for Opus |

---

## Part 5: Best Practices

### Match Model to Task

| Task | Model |
|------|-------|
| Typo fixes | Haiku |
| Simple edits | Sonnet |
| Architecture | Opus |
| Big features | opusplan |

### Check Current Model

```bash
/status
```

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `/model <name>` | Switch model |
| `/status` | Check current model |

---

## Sources

- [Model Configuration](https://code.claude.com/docs/en/model-config.md)
