# Tutorial 51: Unix Utility Integration

> **Use Claude Code as part of your Unix pipeline**

## Overview

Claude Code works as a Unix-style utility, fitting into pipes and scripts.

---

## Part 1: Pipe Input

### Pipe File Content

```bash
cat file.py | claude -p "Review this code"
```

### Pipe Command Output

```bash
npm test 2>&1 | claude -p "Explain these test failures"
```

---

## Part 2: Pipe Output

### Save to File

```bash
claude -p "Generate a README" > README.md
```

### Chain Commands

```bash
claude -p "List all TODO comments" | grep URGENT
```

---

## Part 3: Structured Output

### JSON Format

```bash
claude -p "Analyze @file.ts" --output-format json | jq '.issues'
```

---

## Part 4: Scripts

### Verification Script

```bash
#!/bin/bash
# pre-commit hook
claude -p "Review staged changes" --permission-mode plan
```

### Batch Processing

```bash
for file in src/*.ts; do
  claude -p "Add JSDoc to @$file"
done
```

---

## Quick Reference

### Pipe Commands

| Pattern | Example |
|---------|---------|
| Input | `cat file \| claude -p "..."` |
| Output | `claude -p "..." > file` |
| Chain | `claude -p "..." \| grep X` |
| JSON | `claude ... --output-format json \| jq` |

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
- [CLI Reference](https://code.claude.com/docs/en/cli-reference.md)
