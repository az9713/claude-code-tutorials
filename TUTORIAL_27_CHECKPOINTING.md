# Tutorial 27: Context Checkpointing

> **Automatically track and rewind Claude's edits to quickly recover from unwanted changes**

## Overview

Checkpointing automatically captures the state of your code before each edit, allowing you to quickly undo changes and rewind to previous states if anything gets off track.

### What You'll Learn

- How checkpoints work automatically
- Rewinding code and conversation state
- Common use cases for checkpointing
- Limitations and best practices

### Prerequisites

- Claude Code installed
- Basic understanding of Claude Code file editing

---

## Part 1: How Checkpoints Work

### Automatic Tracking

Claude Code automatically tracks every code change:

| Feature | Description |
|---------|-------------|
| **Every prompt creates a checkpoint** | Before Claude edits, state is saved |
| **Persists across sessions** | Access checkpoints in resumed conversations |
| **Automatic cleanup** | Removed after 30 days (configurable) |
| **Zero configuration** | Works out of the box |

### What Gets Tracked

```
Tracked:
├── File edits via Claude's editing tools
├── File writes via Claude's write tool
└── All modifications in current session

NOT Tracked:
├── Bash command modifications (rm, mv, cp)
├── Manual changes outside Claude Code
└── Changes from other concurrent sessions
```

---

## Part 2: Rewinding Changes

### Quick Rewind

Press `Esc` twice (`Esc` + `Esc`) to open the rewind menu.

Alternatively, use the `/rewind` command:

```bash
/rewind
```

### Rewind Options

| Option | What It Does |
|--------|--------------|
| **Conversation only** | Rewind to a user message, keep code changes |
| **Code only** | Revert file changes, keep conversation |
| **Both** | Restore both code and conversation |

### Step-by-Step Rewind

1. Press `Esc` + `Esc` or type `/rewind`
2. Select the checkpoint to rewind to
3. Choose what to restore (conversation, code, or both)
4. Confirm the rewind

---

## Part 3: Common Use Cases

### Exploring Alternatives

Try different implementation approaches without risk:

```bash
You: Implement the caching layer using Redis

# Claude implements Redis caching

You: Actually, let's try with in-memory caching instead

# Press Esc+Esc, rewind code, try different approach

You: Implement using in-memory LRU cache instead
```

### Recovering from Mistakes

Quickly undo changes that broke functionality:

```bash
You: Refactor the authentication module

# Claude makes changes, but tests fail

# Press Esc+Esc, rewind to before the refactor
# Give more specific instructions

You: Refactor authentication but preserve the session handling
```

### Iterating on Features

Experiment with variations knowing you can revert:

```bash
You: Add dark mode support

# Review implementation, want changes

# Rewind code only, keep conversation context

You: Let's keep the toggle but use CSS variables instead of inline styles
```

---

## Part 4: Checkpoint Management

### Viewing Checkpoint History

The rewind menu shows:
- Timestamp of each checkpoint
- The user prompt that triggered it
- Summary of changes made

### Selective Restoration

You can restore different aspects independently:

**Scenario:** You liked Claude's explanation but not the code changes

```bash
/rewind
# Select checkpoint
# Choose "Conversation only" - code stays as-is
```

**Scenario:** Code was fine but conversation went off-track

```bash
/rewind
# Select checkpoint
# Choose "Code only" - conversation resets
```

---

## Part 5: Limitations

### Bash Command Changes NOT Tracked

These modifications cannot be undone through rewind:

```bash
rm file.txt           # File deletion
mv old.txt new.txt    # File rename
cp source.txt dest.txt # File copy
sed -i 's/foo/bar/g' *.py  # In-place editing
```

Only direct file edits through Claude's file editing tools are tracked.

### External Changes NOT Tracked

- Manual edits you make outside Claude Code
- Changes from other concurrent Claude Code sessions
- IDE or editor modifications

### Not a Replacement for Version Control

| Checkpoints | Git |
|-------------|-----|
| Session-level recovery | Permanent history |
| Quick local undo | Collaboration & branching |
| Automatic | Manual commits |
| 30-day retention | Indefinite |

**Think of checkpoints as "local undo" and Git as "permanent history".**

---

## Part 6: Best Practices

### 1. Commit Before Major Changes

```bash
# Commit current state to Git
!git add -A && git commit -m "Before major refactor"

# Then proceed with Claude's changes
You: Refactor the entire data layer
```

### 2. Use Checkpoints for Exploration

Checkpoints are perfect for:
- Trying out ideas
- A/B testing implementations
- Iterating on design

### 3. Combine with Git for Safety

```bash
# Create a branch for exploration
!git checkout -b experiment/new-feature

# Use Claude with checkpoints for quick iterations
# Once happy, commit and merge
```

### 4. Rewind Early, Rewind Often

Don't wait until things are completely broken:
- Notice something slightly off? Rewind immediately
- Easier to fix from a clean state

### 5. Be Specific After Rewinding

When you rewind and try again, give Claude more specific guidance:

**Before rewind:**
```bash
You: Improve the API
```

**After rewind, be more specific:**
```bash
You: Improve the API by:
1. Adding input validation
2. Implementing proper error responses
3. Adding rate limiting
But do NOT change the endpoint paths
```

---

## Part 7: Advanced Patterns

### Pattern 1: Checkpoint Sandwich

```bash
# Create Git commit (permanent save point)
!git commit -m "Save point"

# Experiment freely with checkpoints
You: Try implementing with pattern A

# Rewind if needed, try again
/rewind

You: Try implementing with pattern B

# Once happy, commit again
!git add -A && git commit -m "Final implementation"
```

### Pattern 2: Incremental Checkpointing

Break large tasks into checkpoint-recoverable steps:

```bash
You: Step 1: Create the database schema
# Checkpoint created

You: Step 2: Implement the models
# Checkpoint created

You: Step 3: Add the API endpoints
# If this fails, rewind to Step 2 checkpoint
```

### Pattern 3: Review and Rewind

```bash
You: Implement the feature

# Claude implements

# Review the changes carefully
# If not satisfied...

/rewind

# Give refined instructions based on what you saw
You: Implement the feature but:
- Use async/await instead of promises
- Add better error messages
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `Esc` + `Esc` | Open rewind menu |
| `/rewind` | Open rewind menu (alternative) |

### Rewind Options

| Option | Code State | Conversation State |
|--------|------------|-------------------|
| Conversation only | Unchanged | Reverted |
| Code only | Reverted | Unchanged |
| Both | Reverted | Reverted |

### What's Tracked

| Tracked | Not Tracked |
|---------|-------------|
| Claude's file edits | Bash command changes |
| Claude's file writes | Manual external edits |
| Session-based changes | Other session changes |

---

## Sources

- [Checkpointing - Official Documentation](https://code.claude.com/docs/en/checkpointing.md)
- [Interactive Mode](https://code.claude.com/docs/en/interactive-mode.md)
- [Slash Commands](https://code.claude.com/docs/en/slash-commands.md)

---

## Next Steps

1. **Try the rewind feature** - Make some changes, then rewind
2. **Practice recovery** - Intentionally create bad changes and recover
3. **Combine with Git** - Use checkpoints alongside version control
