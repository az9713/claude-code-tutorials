# Tutorial 61: Terminal Customization & Protocols

> **Optimize Claude Code for your terminal emulator with protocol support and configuration**

## Table of Contents

1. [Overview](#overview)
2. [Terminal Detection](#terminal-detection)
3. [OSC 8 Hyperlinks](#osc-8-hyperlinks)
4. [Kitty Keyboard Protocol](#kitty-keyboard-protocol)
5. [Terminal-Specific Setup](#terminal-specific-setup)
6. [Input Methods & IME](#input-methods--ime)
7. [Shift+Enter Support](#shiftenter-support)
8. [Statusline Configuration](#statusline-configuration)
9. [Color and Theme Support](#color-and-theme-support)
10. [Troubleshooting](#troubleshooting)
11. [Quick Reference](#quick-reference)

---

## Overview

Claude Code adapts to different terminal emulators to provide the best possible experience. This tutorial covers terminal protocols, customization options, and terminal-specific configurations.

### Why Terminal Configuration Matters

Different terminals support different features:

| Feature | Basic Terminal | Modern Terminal |
|---------|---------------|-----------------|
| Clickable links | ❌ | ✓ |
| Key disambiguation | Limited | Full |
| Unicode rendering | Varies | Full |
| Image display | ❌ | Some |
| True color | Some | ✓ |

### Supported Terminals

**Full Feature Support:**
- iTerm2 (macOS)
- Kitty (cross-platform)
- WezTerm (cross-platform)
- Ghostty (macOS/Linux)
- Alacritty (cross-platform)
- Windows Terminal

**Basic Support:**
- macOS Terminal.app
- GNOME Terminal
- Konsole
- xterm

---

## Terminal Detection

### How Claude Code Detects Your Terminal

Claude Code checks environment variables to identify your terminal:

```bash
# Check what Claude Code sees
echo $TERM
echo $TERM_PROGRAM
echo $COLORTERM
```

| Variable | Example Values |
|----------|---------------|
| `TERM` | `xterm-256color`, `screen-256color` |
| `TERM_PROGRAM` | `iTerm.app`, `WezTerm`, `vscode` |
| `COLORTERM` | `truecolor`, `24bit` |

### Manual Terminal Configuration

Override detection in settings:

```json
{
  "terminal": {
    "type": "kitty",
    "features": {
      "hyperlinks": true,
      "kittyKeyboard": true,
      "trueColor": true
    }
  }
}
```

---

## OSC 8 Hyperlinks

### What Are OSC 8 Hyperlinks?

OSC 8 is a terminal escape sequence that creates clickable links. Instead of displaying raw URLs, text becomes clickable:

```
Before: See https://docs.example.com/api/endpoint
After:  See API Documentation (clickable)
```

### Supported Since v2.1.2

Claude Code v2.1.2+ generates OSC 8 hyperlinks for:
- File paths in output
- URLs in responses
- Error locations
- Documentation references

### Enabling Hyperlinks

Most modern terminals support OSC 8 automatically. Verify support:

```bash
# Test OSC 8 support
printf '\e]8;;https://example.com\aClick here\e]8;;\a\n'
# If "Click here" is clickable, OSC 8 works
```

### Terminal-Specific Setup

**iTerm2:**
Already enabled by default.

**Kitty:**
Already enabled by default.

**WezTerm:**
Already enabled by default.

**GNOME Terminal:**
Requires VTE 0.50+ (Ubuntu 18.04+).

**Windows Terminal:**
Already enabled by default.

### Disabling Hyperlinks

If hyperlinks cause issues:

```json
{
  "terminal": {
    "features": {
      "hyperlinks": false
    }
  }
}
```

---

## Kitty Keyboard Protocol

### What Is It?

The Kitty keyboard protocol provides unambiguous key reporting, solving issues where terminals can't distinguish between:
- `Ctrl+I` vs `Tab`
- `Ctrl+M` vs `Enter`
- `Ctrl+[` vs `Escape`
- Modified keys with modifiers

### Supported Since v2.1.6 / v2.1.9

Claude Code v2.1.6 added initial support, with improvements in v2.1.9.

### Benefits

| Problem | Without Protocol | With Protocol |
|---------|------------------|---------------|
| Ctrl+I | Sends Tab | Sends Ctrl+I |
| Ctrl+M | Sends Enter | Sends Ctrl+M |
| Ctrl+Shift+V | May not work | Works correctly |
| Alt combinations | Inconsistent | Consistent |

### Supported Terminals

- **Kitty** - Full support
- **WezTerm** - Full support
- **Ghostty** - Full support
- **foot** - Full support
- **iTerm2** - Partial support

### Configuration

Enable in settings:

```json
{
  "terminal": {
    "features": {
      "kittyKeyboard": true
    }
  }
}
```

Or use the setup command:

```bash
/terminal-setup
```

### Verifying Protocol Support

```bash
# In Kitty/WezTerm, check protocol level
kitty +kitten icat --print-window-size 2>/dev/null && echo "Kitty graphics supported"
```

---

## Terminal-Specific Setup

### iTerm2 (macOS)

**Setup:**

1. Enable features in Preferences → Profiles → Terminal:
   - ✓ Enable mouse reporting
   - ✓ Report mouse clicks
   - ✓ Report mouse wheel events

2. Enable Cmd+V paste:
   - Preferences → General → Selection → ✓ Applications in terminal may access clipboard

3. For image support:
   - Preferences → General → Magic → ✓ Enable inline images

**Configuration:**
```json
{
  "terminal": {
    "type": "iterm2",
    "features": {
      "hyperlinks": true,
      "images": true
    }
  }
}
```

### Kitty

**Setup:**

1. Add to `~/.config/kitty/kitty.conf`:
   ```
   enable_audio_bell no
   scrollback_lines 10000

   # Enable keyboard protocol
   # (Usually automatic for Claude Code)
   ```

2. For image support:
   ```
   # Already enabled by default
   ```

**Configuration:**
```json
{
  "terminal": {
    "type": "kitty",
    "features": {
      "hyperlinks": true,
      "kittyKeyboard": true,
      "images": true
    }
  }
}
```

### WezTerm

**Setup:**

Add to `~/.wezterm.lua`:
```lua
local wezterm = require 'wezterm'
local config = {}

-- Enable hyperlinks
config.hyperlink_rules = wezterm.default_hyperlink_rules()

-- Enable keyboard protocol
config.enable_kitty_keyboard = true

-- Enable images
config.enable_kitty_graphics = true

return config
```

**Configuration:**
```json
{
  "terminal": {
    "type": "wezterm",
    "features": {
      "hyperlinks": true,
      "kittyKeyboard": true,
      "images": true
    }
  }
}
```

### Ghostty

**Setup:**

Add to `~/.config/ghostty/config`:
```
# Enable advanced input handling
enable-keyboard-protocol = true

# Font configuration
font-family = "JetBrains Mono"
font-size = 14
```

**Configuration:**
```json
{
  "terminal": {
    "type": "ghostty",
    "features": {
      "hyperlinks": true,
      "kittyKeyboard": true
    }
  }
}
```

### Alacritty

**Setup:**

Add to `~/.config/alacritty/alacritty.yml`:
```yaml
hints:
  enabled:
    - regex: "(https?://)[^\u0000-\u001F\u007F-\u009F<>\"\\s{-}\\^⟨⟩`]+"
      command: xdg-open
      post_processing: true
      mouse:
        enabled: true
```

**Note:** Alacritty has limited protocol support compared to Kitty/WezTerm.

### Windows Terminal

**Setup:**

1. Open Settings (Ctrl+,)
2. Enable in Profile → Advanced:
   - ✓ Automatically detect URLs and make them clickable

3. Add to `settings.json`:
```json
{
  "profiles": {
    "defaults": {
      "experimental.retroTerminalEffect": false,
      "useAcrylic": false
    }
  }
}
```

### VS Code Integrated Terminal

**Setup:**

Add to `settings.json`:
```json
{
  "terminal.integrated.experimentalLinkProvider": true,
  "terminal.integrated.detectLinks": true,
  "terminal.integrated.enablePersistentSessions": true
}
```

---

## Input Methods & IME

### CJK Input Support

Claude Code v2.1.x includes improved support for:
- Chinese (Pinyin, Wubi, etc.)
- Japanese (Hiragana, Katakana, IME)
- Korean (Hangul)

### Configuration

```json
{
  "terminal": {
    "input": {
      "enableIME": true,
      "compositionTimeout": 500
    }
  }
}
```

### Word Navigation with CJK

The `language` setting affects word navigation:

```json
{
  "language": "ja"  // Japanese word boundaries
}
```

### Troubleshooting IME

**Issue:** Characters appear incorrectly during composition

**Solution:**
1. Ensure terminal supports Unicode properly
2. Use a font with CJK coverage (e.g., Noto Sans CJK)
3. Set proper locale:
   ```bash
   export LANG=en_US.UTF-8
   export LC_ALL=en_US.UTF-8
   ```

---

## Shift+Enter Support

### Multi-line Input (v2.1.0)

Shift+Enter creates a new line without submitting:

```
You: This is line 1
     This is line 2 (Shift+Enter)
     This is line 3 (Shift+Enter)
     [Enter to submit all lines]
```

### Terminal Requirements

| Terminal | Shift+Enter Support |
|----------|---------------------|
| iTerm2 | ✓ Native |
| Kitty | ✓ With protocol |
| WezTerm | ✓ With protocol |
| Ghostty | ✓ With protocol |
| Windows Terminal | ✓ Native |
| GNOME Terminal | Requires setup |
| macOS Terminal | Limited |

### Enabling in GNOME Terminal

Add to `~/.inputrc`:
```
"\e[13;2u": "\n"
```

### Alternative: Explicit Newlines

If Shift+Enter doesn't work, use explicit markers:

```
You: Line 1 \
     Line 2 \
     Line 3
```

---

## Statusline Configuration

### Statusline Overview

Claude Code displays a statusline showing:
- Current model
- Token usage
- Session status
- Active operations

### Configuration Options (v2.1.x)

```json
{
  "statusline": {
    "enabled": true,
    "position": "bottom",
    "showModel": true,
    "showTokens": true,
    "showTurnDuration": true,
    "showSessionId": false
  }
}
```

### Hiding Turn Duration (v2.1.7)

```json
{
  "showTurnDuration": false
}
```

### Custom Statusline Format

```json
{
  "statusline": {
    "format": "{model} | {tokens} tokens | {status}"
  }
}
```

### Terminal-Specific Statusline Issues

**Issue:** Statusline overlaps with content

**Solution:**
```json
{
  "statusline": {
    "enabled": true,
    "reserveLines": 2
  }
}
```

---

## Color and Theme Support

### True Color Detection

Claude Code detects true color (24-bit) support via:
- `COLORTERM=truecolor`
- `COLORTERM=24bit`
- Terminal-specific detection

### Force True Color

```json
{
  "terminal": {
    "features": {
      "trueColor": true
    }
  }
}
```

Or via environment:
```bash
export COLORTERM=truecolor
claude
```

### Theme Support

Claude Code respects terminal themes:

```json
{
  "theme": "auto",  // Follows terminal dark/light mode
  "theme": "dark",  // Force dark
  "theme": "light"  // Force light
}
```

### Syntax Highlighting

Code blocks use terminal-appropriate colors:

```json
{
  "highlighting": {
    "enabled": true,
    "style": "monokai"  // Or: github, dracula, nord, etc.
  }
}
```

---

## Troubleshooting

### Terminal Not Detected Correctly

**Symptom:** Features not working as expected

**Solution:**
```json
{
  "terminal": {
    "type": "kitty",  // Manually specify
    "features": {
      "hyperlinks": true,
      "kittyKeyboard": true
    }
  }
}
```

### Hyperlinks Not Clickable

**Check:**
1. Terminal supports OSC 8
2. Feature is enabled in terminal settings
3. Not running inside tmux/screen (may need passthrough)

**For tmux:**
Add to `~/.tmux.conf`:
```
set -g allow-passthrough on
```

### Keys Not Working Correctly

**Symptom:** Ctrl+C doesn't cancel, special keys misbehave

**Solutions:**
1. Enable Kitty keyboard protocol if supported
2. Check `$TERM` is set correctly
3. Run `/terminal-setup` for guided configuration

### Display Corruption

**Symptom:** Garbled output, misaligned text

**Solutions:**
1. Reset terminal: `reset` or `tput reset`
2. Check encoding: `locale` should show UTF-8
3. Use a font with complete Unicode coverage
4. Disable problematic features:
   ```json
   {
     "terminal": {
       "features": {
         "hyperlinks": false,
         "images": false
       }
     }
   }
   ```

### Performance Issues

**Symptom:** Slow rendering, lag

**Solutions:**
1. Disable images:
   ```json
   { "terminal": { "features": { "images": false } } }
   ```
2. Reduce scrollback in terminal settings
3. Use GPU-accelerated terminal (Kitty, Alacritty, WezTerm)

---

## Quick Reference

### /terminal-setup Command

Run for guided terminal configuration:
```bash
/terminal-setup
```

This command:
1. Detects your terminal
2. Tests feature support
3. Recommends optimal settings
4. Applies configuration

### Feature Detection

| Feature | Environment Check | Test Command |
|---------|------------------|--------------|
| True color | `$COLORTERM` | `printf '\e[38;2;255;0;0mRED\e[0m\n'` |
| OSC 8 links | - | `printf '\e]8;;URL\aText\e]8;;\a\n'` |
| Kitty protocol | `$TERM=xterm-kitty` | Check for modifier keys |
| Unicode | `$LANG` | `echo "日本語 한국어 中文"` |

### Terminal Comparison

| Terminal | Hyperlinks | Kitty KB | Images | True Color |
|----------|------------|----------|--------|------------|
| iTerm2 | ✓ | Partial | ✓ | ✓ |
| Kitty | ✓ | ✓ | ✓ | ✓ |
| WezTerm | ✓ | ✓ | ✓ | ✓ |
| Ghostty | ✓ | ✓ | ✓ | ✓ |
| Alacritty | ✓ | ❌ | ❌ | ✓ |
| Win Terminal | ✓ | Partial | ❌ | ✓ |
| GNOME Term | ✓ | ❌ | ❌ | ✓ |
| Terminal.app | ❌ | ❌ | ❌ | ❌ |

### Configuration Template

```json
{
  "terminal": {
    "type": "auto",
    "features": {
      "hyperlinks": true,
      "kittyKeyboard": true,
      "trueColor": true,
      "images": true
    },
    "input": {
      "enableIME": true
    }
  },
  "statusline": {
    "enabled": true,
    "showModel": true,
    "showTokens": true,
    "showTurnDuration": true
  },
  "theme": "auto"
}
```

### Common Environment Variables

```bash
# Terminal identification
export TERM=xterm-256color
export TERM_PROGRAM=iTerm.app
export COLORTERM=truecolor

# Locale (for Unicode)
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8

# Claude Code specific
export CLAUDE_CODE_SHELL=/bin/zsh
```

---

## Sources

- [Claude Code Terminal Configuration](https://code.claude.com/docs/en/terminal-config.md)
- [Claude Code Changelog v2.1.2, v2.1.6, v2.1.9](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Kitty Keyboard Protocol](https://sw.kovidgoyal.net/kitty/keyboard-protocol/)
- [OSC 8 Hyperlinks Spec](https://gist.github.com/egmontkob/eb114294efbcd5adb1944c9f3cb5feda)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
