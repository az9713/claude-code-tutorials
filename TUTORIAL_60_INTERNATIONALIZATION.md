# Tutorial 60: Internationalization & Localization

> **Multi-language support, CJK input handling, and Unicode configuration in Claude Code**

## Table of Contents

1. [Overview](#overview)
2. [Language Setting](#language-setting)
3. [CJK Input Support](#cjk-input-support)
4. [IME Configuration](#ime-configuration)
5. [Unicode Handling](#unicode-handling)
6. [Word Navigation](#word-navigation)
7. [Terminal Configuration](#terminal-configuration)
8. [Font Requirements](#font-requirements)
9. [Troubleshooting](#troubleshooting)
10. [Quick Reference](#quick-reference)

---

## Overview

Claude Code v2.1.x includes improved internationalization support, particularly for CJK (Chinese, Japanese, Korean) languages. This tutorial covers configuration for multilingual use.

### Key Features

| Feature | Version | Description |
|---------|---------|-------------|
| `language` setting | v2.1.0 | Set response language |
| CJK input | v2.1.x | Improved IME support |
| Word navigation | v2.1.x | Language-aware boundaries |
| Unicode display | v2.1.x | Better emoji/symbol handling |

---

## Language Setting

### Configuration (v2.1.0)

Set your preferred response language:

```json
{
  "language": "ja"
}
```

### Supported Language Codes

| Code | Language |
|------|----------|
| `en` | English (default) |
| `zh` | Chinese |
| `ja` | Japanese |
| `ko` | Korean |
| `es` | Spanish |
| `fr` | French |
| `de` | German |
| `pt` | Portuguese |
| `it` | Italian |
| `ru` | Russian |
| `ar` | Arabic |
| `hi` | Hindi |

### Effects of Language Setting

1. **Response language**: Claude responds in specified language
2. **Word boundaries**: Navigation respects language-specific rules
3. **Date/time formats**: Localized when applicable
4. **Help text**: Some UI elements may be localized

### Example: Japanese Configuration

```json
{
  "language": "ja",
  "terminal": {
    "input": {
      "enableIME": true
    }
  }
}
```

---

## CJK Input Support

### Chinese Input

```bash
# Set locale
export LANG=zh_CN.UTF-8

# Configure IME
# Varies by system - common options:
# - ibus
# - fcitx
# - macOS built-in
```

**Supported input methods:**
- Pinyin
- Wubi
- Cangjie
- Zhuyin

### Japanese Input

```bash
# Set locale
export LANG=ja_JP.UTF-8

# IME options:
# - macOS: Built-in Japanese
# - Linux: ibus-anthy, fcitx-mozc
# - Windows: Microsoft IME
```

**Supported input methods:**
- Hiragana
- Katakana
- Romaji conversion

### Korean Input

```bash
# Set locale
export LANG=ko_KR.UTF-8

# IME options:
# - macOS: Built-in Korean
# - Linux: ibus-hangul, fcitx-hangul
# - Windows: Microsoft IME
```

---

## IME Configuration

### Enable IME Support

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

### IME Behavior

| Setting | Default | Description |
|---------|---------|-------------|
| `enableIME` | `true` | Enable input method support |
| `compositionTimeout` | `500` | Delay before composition (ms) |

### Composition Display

During IME composition:
```
You: 私は[入力中]_
```

After confirmation:
```
You: 私はコードを書いています
```

### Troubleshooting IME

**Issue:** Characters appear incorrectly during composition

**Solutions:**
1. Ensure terminal supports IME
2. Use compatible font
3. Check locale settings

```bash
# Verify locale
locale

# Should show UTF-8 encoding
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
```

---

## Unicode Handling

### Display Requirements

Claude Code supports full Unicode including:
- CJK characters (Chinese, Japanese, Korean)
- Emojis 😀
- Mathematical symbols ∑∫∂
- Box drawing ┌─┐
- Arrows →←↑↓

### Unicode Width Handling

Different character widths:

| Type | Width | Example |
|------|-------|---------|
| ASCII | 1 | `a` |
| CJK | 2 | `日` |
| Emoji | 2 | `😀` |
| Combining | 0 | `̈` (diaeresis) |

### Configuration for Wide Characters

```json
{
  "terminal": {
    "unicodeWidth": "auto"
  }
}
```

Options:
- `auto` - Detect from environment
- `ambiguous-wide` - Treat ambiguous as wide
- `ambiguous-narrow` - Treat ambiguous as narrow

---

## Word Navigation

### Language-Aware Navigation

With `language` setting, word navigation respects language rules:

**English (default):**
```
the|quick|brown|fox
    ← w    →
```

**Japanese:**
```
今日は|良い|天気|です
     ← w   →
```

**Chinese:**
```
今天|天气|很|好
   ← w  →
```

### Vim Mode Navigation

```
w - Next word (language-aware)
b - Previous word (language-aware)
e - End of word (language-aware)
```

### Configuration

```json
{
  "language": "ja",
  "terminal": {
    "wordBoundaries": "language-aware"
  }
}
```

---

## Terminal Configuration

### Locale Setup

**macOS:**
```bash
# ~/.zshrc or ~/.bashrc
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

**Linux:**
```bash
# Generate locale
sudo locale-gen en_US.UTF-8
sudo locale-gen ja_JP.UTF-8

# Set default
sudo update-locale LANG=en_US.UTF-8
```

**Windows:**
```powershell
# PowerShell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$env:PYTHONIOENCODING = "utf-8"
```

### Terminal-Specific Setup

**iTerm2 (macOS):**
1. Preferences → Profiles → Terminal
2. Set Character Encoding: UTF-8
3. Enable: Report terminal type as xterm-256color

**Windows Terminal:**
1. Settings → Profiles → Defaults
2. Starting directory: Your choice
3. Font: Use CJK-compatible font

**GNOME Terminal:**
1. Preferences → Profile → Compatibility
2. Encoding: UTF-8

---

## Font Requirements

### Recommended Fonts

| Platform | Font | CJK Support |
|----------|------|-------------|
| Cross-platform | Noto Sans Mono CJK | Excellent |
| Cross-platform | Source Han Code JP | Excellent |
| macOS | SF Mono | With fallback |
| Windows | MS Gothic | Japanese |
| Windows | Microsoft YaHei | Chinese |

### Font Configuration

**macOS iTerm2:**
1. Preferences → Profiles → Text
2. Font: Noto Sans Mono CJK JP
3. Non-ASCII Font: Same or compatible

**VS Code Terminal:**
```json
{
  "terminal.integrated.fontFamily": "'Noto Sans Mono CJK JP', 'Fira Code', monospace"
}
```

### Emoji Font Fallback

For emoji display, ensure fallback font:

```json
{
  "terminal": {
    "fontFamily": "JetBrains Mono, Apple Color Emoji, Segoe UI Emoji, monospace"
  }
}
```

---

## Troubleshooting

### Characters Display as Boxes

**Cause:** Missing font support

**Solution:**
```bash
# Install CJK fonts

# macOS (with Homebrew)
brew install --cask font-noto-sans-cjk

# Ubuntu/Debian
sudo apt install fonts-noto-cjk

# Fedora
sudo dnf install google-noto-sans-cjk-fonts
```

### IME Not Working

**Cause:** Terminal not supporting IME

**Solution:**
1. Use IME-compatible terminal (iTerm2, Windows Terminal)
2. Check IME is enabled system-wide
3. Verify terminal IME settings

### Garbled Text Output

**Cause:** Encoding mismatch

**Solution:**
```bash
# Check current encoding
echo $LANG

# Fix encoding
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

### Wide Characters Misaligned

**Cause:** Terminal width calculation wrong

**Solution:**
```json
{
  "terminal": {
    "unicodeWidth": "ambiguous-wide"
  }
}
```

### Input Characters Doubled

**Cause:** IME composition issue

**Solution:**
```json
{
  "terminal": {
    "input": {
      "compositionTimeout": 1000
    }
  }
}
```

---

## Quick Reference

### Settings

```json
{
  "language": "ja",
  "terminal": {
    "input": {
      "enableIME": true,
      "compositionTimeout": 500
    },
    "unicodeWidth": "auto",
    "wordBoundaries": "language-aware"
  }
}
```

### Environment Variables

```bash
# Locale
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8

# Or for specific languages
export LANG=ja_JP.UTF-8  # Japanese
export LANG=zh_CN.UTF-8  # Chinese (Simplified)
export LANG=ko_KR.UTF-8  # Korean
```

### Language Codes

| Code | Language | Script |
|------|----------|--------|
| `en` | English | Latin |
| `ja` | Japanese | Hiragana/Katakana/Kanji |
| `zh` | Chinese | Hanzi |
| `zh-TW` | Chinese (Traditional) | Hanzi |
| `ko` | Korean | Hangul |

### Font Installation Commands

```bash
# macOS
brew install --cask font-noto-sans-cjk

# Ubuntu/Debian
sudo apt install fonts-noto-cjk fonts-noto-color-emoji

# Fedora
sudo dnf install google-noto-sans-cjk-fonts google-noto-emoji-fonts

# Windows (PowerShell - download Noto fonts)
# Install manually from https://fonts.google.com/noto
```

### Common IME Shortcuts

| Platform | Toggle IME |
|----------|------------|
| macOS | Ctrl+Space or ⌘+Space |
| Windows | Alt+Shift or Win+Space |
| Linux (ibus) | Ctrl+Space |
| Linux (fcitx) | Ctrl+Space |

---

## Sources

- [Claude Code Settings](https://code.claude.com/docs/en/settings.md)
- [Claude Code Changelog v2.1.0](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Unicode Technical Reports](https://unicode.org/reports/)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
