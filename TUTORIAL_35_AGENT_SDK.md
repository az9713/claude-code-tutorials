# Tutorial 35: Agent SDK

> **Build custom agents programmatically with Claude Code**

## Overview

The Agent SDK enables programmatic control of Claude Code for custom automation and integrations.

---

## Part 1: Headless Mode Basics

### Basic Usage

```bash
claude -p "Your prompt" --print
```

Key flags:
- `-p` / `--print`: Non-interactive mode
- `--output-format json`: Structured output
- `--permission-mode acceptEdits`: Auto-approve changes

### JSON Output

```bash
claude -p "List all TODO comments" \
  --output-format json \
  --print
```

Returns structured JSON with results.

---

## Part 2: Common Patterns

### Get Structured Output

```bash
echo "Analyze src/api/ and return a JSON summary" | \
  claude -p - --output-format json
```

### Auto-Approve Tools

```bash
claude -p "Fix lint errors" \
  --permission-mode acceptEdits \
  --allowedTools "Read(*) Edit(*) Write(*)"
```

### Continue Conversations

```bash
# Start conversation, save session ID
SESSION=$(claude -p "Start analysis" --output-format json | jq -r '.sessionId')

# Continue later
claude --resume "$SESSION" -p "Now implement fixes"
```

---

## Part 3: System Prompt Customization

### Append to System Prompt

```bash
claude -p "Review code" \
  --append-system-prompt "Focus only on security issues"
```

### Custom Prompt Files

```bash
claude -p "$(cat custom-prompt.txt)"
```

---

## Part 4: Integration Examples

### Git Hooks

```bash
#!/bin/bash
# .git/hooks/pre-commit

claude -p "Review staged changes for issues" \
  --print \
  --permission-mode plan

if [ $? -ne 0 ]; then
  echo "Issues found, commit blocked"
  exit 1
fi
```

### CI/CD Pipeline

```yaml
- name: Claude Analysis
  run: |
    result=$(claude -p "Analyze for security issues" --output-format json)
    echo "$result" > analysis.json
```

### Custom Scripts

```bash
#!/bin/bash
# auto-review.sh

for file in "$@"; do
  claude -p "Review $file for best practices" --print
done
```

---

## Part 5: Tool Permissions

### Restrict Tools

```bash
# Read-only analysis
claude -p "Analyze code" \
  --allowedTools "Read(*) Search(*)"

# Full editing
claude -p "Fix bugs" \
  --allowedTools "Bash(*) Read(*) Edit(*) Write(*)"
```

### Permission Modes

| Mode | Description |
|------|-------------|
| `default` | Ask for each action |
| `acceptEdits` | Auto-accept file changes |
| `bypassPermissions` | Skip all prompts |
| `plan` | Read-only analysis |

---

## Part 6: Error Handling

### Check Exit Codes

```bash
if claude -p "Run tests" --print; then
  echo "Success"
else
  echo "Failed with code $?"
fi
```

### Capture Output

```bash
output=$(claude -p "Analyze" --output-format json 2>&1)
if echo "$output" | jq -e '.error' > /dev/null; then
  echo "Error: $(echo $output | jq -r '.error')"
fi
```

---

## Quick Reference

### Key Flags

| Flag | Purpose |
|------|---------|
| `-p <prompt>` | Non-interactive prompt |
| `--print` | Print mode |
| `--output-format json` | JSON output |
| `--permission-mode <mode>` | Set permissions |
| `--allowedTools <list>` | Restrict tools |
| `--append-system-prompt` | Add to system prompt |
| `--resume <id>` | Continue session |
| `--max-turns <n>` | Limit iterations |

---

## Sources

- [Headless Mode](https://code.claude.com/docs/en/headless.md)
- [CLI Reference](https://code.claude.com/docs/en/cli-reference.md)
