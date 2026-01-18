# Tutorial 53: Keyboard Shortcuts

> **Master Claude Code keyboard shortcuts**

## Overview

Master productivity with keyboard shortcuts.

---

## Part 1: Terminal Shortcuts

### Session Control

| Shortcut | Action |
|----------|--------|
| `Esc` | Cancel current operation |
| `Esc + Esc` | Open rewind menu |
| `Ctrl+C` | Interrupt |
| `Ctrl+D` | Exit session |

### Bash Shortcut

```bash
!npm test    # Runs npm test directly
!pwd         # Any bash command
```

---

## Part 2: VS Code Shortcuts

| Shortcut | Mac | Windows/Linux |
|----------|-----|---------------|
| Toggle focus | `Cmd+Esc` | `Ctrl+Esc` |
| New tab | `Cmd+Shift+Esc` | `Ctrl+Shift+Esc` |
| New conversation | `Cmd+N` | `Ctrl+N` |
| Insert @-mention | `Alt+K` | `Alt+K` |

---

## Part 3: JetBrains Shortcuts

| Shortcut | Mac | Windows/Linux |
|----------|-----|---------------|
| Open Claude | `Cmd+Esc` | `Ctrl+Esc` |
| File reference | `Cmd+Option+K` | `Ctrl+Alt+K` |

---

## Part 4: Slash Commands

Quick access to features:

| Command | Purpose |
|---------|---------|
| `/help` | Show commands |
| `/compact` | Compress context |
| `/clear` | Reset context |
| `/model` | Change model |
| `/config` | Settings |

---

## Part 5: Vim Mode

### Enabling Vim Mode

Configure in settings:

```json
{
  "inputMode": "vim"
}
```

Or start with vim mode:

```bash
# Not currently a CLI flag, use settings
```

### Basic Vim Motions

| Key | Action |
|-----|--------|
| `h` | Move left |
| `j` | Move down (history) |
| `k` | Move up (history) |
| `l` | Move right |
| `w` | Word forward |
| `b` | Word backward |
| `e` | End of word |
| `0` | Start of line |
| `$` | End of line |
| `^` | First non-whitespace |

### Mode Switching

| Key | From → To |
|-----|-----------|
| `i` | Normal → Insert (before cursor) |
| `a` | Normal → Insert (after cursor) |
| `I` | Normal → Insert (start of line) |
| `A` | Normal → Insert (end of line) |
| `Esc` | Insert → Normal |

### Advanced Vim Motions (v2.1.0)

| Key | Action |
|-----|--------|
| `;` | Repeat last `f`/`t`/`F`/`T` forward |
| `,` | Repeat last `f`/`t`/`F`/`T` backward |
| `f{char}` | Find character forward |
| `F{char}` | Find character backward |
| `t{char}` | Till character forward |
| `T{char}` | Till character backward |

### Yank and Paste (v2.1.0)

| Key | Action |
|-----|--------|
| `y` | Yank (copy) selection |
| `yy` | Yank entire line |
| `yw` | Yank word |
| `y$` | Yank to end of line |
| `p` | Paste after cursor |
| `P` | Paste before cursor |

### Text Objects (v2.1.0)

| Key | Action |
|-----|--------|
| `iw` | Inner word |
| `aw` | A word (includes space) |
| `i"` | Inside double quotes |
| `a"` | Around double quotes |
| `i(` | Inside parentheses |
| `a(` | Around parentheses |
| `i{` | Inside braces |
| `a{` | Around braces |

**Examples:**
- `diw` - Delete inner word
- `ci"` - Change inside quotes
- `ya(` - Yank around parentheses

### Indentation (v2.1.0)

| Key | Action |
|-----|--------|
| `>>` | Indent line right |
| `<<` | Indent line left |
| `>` | Indent selection |
| `<` | Unindent selection |

### Line Operations (v2.1.0)

| Key | Action |
|-----|--------|
| `J` | Join lines |
| `dd` | Delete line |
| `cc` | Change line |
| `D` | Delete to end of line |
| `C` | Change to end of line |

---

## Part 6: New Shortcuts (v2.0.x - v2.1.x)

### Alt Key Shortcuts

| Shortcut | Action | Version |
|----------|--------|---------|
| `Alt+T` | Toggle thinking display | v2.0.72 |
| `Alt+P` | Switch model | v2.0.65 |
| `Alt+Y` | Yank-pop (paste from kill ring) | v2.0.73 |

### Thinking Toggle (Alt+T)

Press `Alt+T` to show/hide Claude's extended thinking:

```
[Extended thinking hidden - press Alt+T to show]

Or when visible:
<thinking>
Claude's detailed reasoning process...
</thinking>
```

### Model Switching (Alt+P)

Quick model switch during conversation:

```
Alt+P

> Select model:
  claude-opus-4-5-20251101
  claude-sonnet-4-20250514 ←
  claude-haiku-3-5-20241022
```

### Yank-Pop (Alt+Y)

After pasting with `Ctrl+Y`, cycle through kill ring:

```
Ctrl+Y  → Paste most recent
Alt+Y   → Replace with previous item
Alt+Y   → Replace with older item
...
```

### Plan Mode Shortcuts

| Shortcut | Action |
|----------|--------|
| `Shift+Tab` | Accept plan edits |

### Dialog Navigation (v2.1.0)

| Key | Action |
|-----|--------|
| `↑` / `↓` | Navigate options |
| `Enter` | Select option |
| `Esc` | Cancel dialog |
| `Tab` | Next field |
| `Shift+Tab` | Previous field |

### Shift+Enter (v2.1.0)

Create multi-line input without submitting:

```
You: Line 1
     Shift+Enter
     Line 2
     Shift+Enter
     Line 3
     Enter (submits all)
```

---

## Part 7: Input Line Editing

### Readline-Style Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+A` | Move to start of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+B` | Move backward one character |
| `Ctrl+F` | Move forward one character |
| `Alt+B` | Move backward one word |
| `Alt+F` | Move forward one word |
| `Ctrl+W` | Delete word backward |
| `Alt+D` | Delete word forward |
| `Ctrl+U` | Delete to start of line |
| `Ctrl+K` | Delete to end of line |
| `Ctrl+Y` | Paste (yank) deleted text |
| `Ctrl+L` | Clear screen |
| `Ctrl+R` | Reverse search history |

### History Navigation

| Shortcut | Action |
|----------|--------|
| `Ctrl+P` / `↑` | Previous command |
| `Ctrl+N` / `↓` | Next command |
| `Ctrl+R` | Search history |
| `Ctrl+G` | Cancel search |

---

## Quick Reference

### Essential Shortcuts

| Action | Terminal | VS Code | JetBrains |
|--------|----------|---------|-----------|
| Cancel | `Esc` | `Esc` | `Esc` |
| Rewind | `Esc+Esc` | — | — |
| Focus | — | `Cmd/Ctrl+Esc` | `Cmd/Ctrl+Esc` |
| Mention | — | `Alt+K` | `Cmd+Option+K` |
| Exit | `Ctrl+D` | — | — |
| Clear | `Ctrl+L` | — | — |

### New v2.0.x-v2.1.x Shortcuts

| Shortcut | Action |
|----------|--------|
| `Alt+T` | Toggle thinking |
| `Alt+P` | Switch model |
| `Alt+Y` | Yank-pop |
| `Shift+Tab` | Accept plan edits |
| `Shift+Enter` | New line (no submit) |

### Vim Mode Summary

| Category | Keys |
|----------|------|
| Movement | `h j k l w b e 0 $ ^` |
| Find char | `f t F T ; ,` |
| Insert | `i a I A` |
| Delete | `d dd D x` |
| Change | `c cc C s` |
| Yank/Paste | `y yy p P` |
| Text Objects | `iw aw i" a" i( a( i{ a{` |
| Indent | `>> << > <` |
| Lines | `J dd cc` |

### Readline Summary

| Category | Keys |
|----------|------|
| Movement | `Ctrl+A E B F`, `Alt+B F` |
| Delete | `Ctrl+W U K`, `Alt+D` |
| History | `Ctrl+P N R` |
| Paste | `Ctrl+Y`, `Alt+Y` |

---

## Sources

- [Interactive Mode](https://code.claude.com/docs/en/interactive-mode.md)
- [VS Code](https://code.claude.com/docs/en/vs-code.md)
- [JetBrains](https://code.claude.com/docs/en/jetbrains.md)
- [Changelog v2.0.65-v2.1.0](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
