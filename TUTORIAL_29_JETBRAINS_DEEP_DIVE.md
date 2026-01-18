# Tutorial 29: JetBrains IDE Integration

> **Use Claude Code with IntelliJ, PyCharm, WebStorm, and other JetBrains IDEs**

## Overview

Claude Code integrates with JetBrains IDEs through a dedicated plugin, providing interactive diff viewing, selection context sharing, and seamless development workflows.

### What You'll Learn

- Installing the JetBrains plugin
- Using Claude Code from your IDE
- Configuration options
- Remote development and WSL setup
- Troubleshooting common issues

### Prerequisites

- A supported JetBrains IDE
- Claude Code CLI installed

---

## Part 1: Supported IDEs

The Claude Code plugin works with most JetBrains IDEs:

| IDE | Language Focus |
|-----|----------------|
| **IntelliJ IDEA** | Java, Kotlin, Scala |
| **PyCharm** | Python |
| **WebStorm** | JavaScript, TypeScript |
| **PhpStorm** | PHP |
| **GoLand** | Go |
| **Android Studio** | Android/Kotlin |
| **Rider** | .NET/C# |
| **RubyMine** | Ruby |
| **CLion** | C/C++ |

---

## Part 2: Installation

### Marketplace Installation

1. Open your JetBrains IDE
2. Go to **Settings** (or **Preferences** on Mac)
3. Navigate to **Plugins**
4. Click **Marketplace** tab
5. Search for "Claude Code"
6. Click **Install**
7. **Restart your IDE**

Or install directly from:
[Claude Code JetBrains Plugin](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-)

> **Note:** After installing, you may need to restart your IDE completely for it to take effect.

### Verify Installation

After restart, you should see:
- Claude Code button in the UI
- Claude Code options in Settings → Tools

---

## Part 3: Features

### Quick Launch

Open Claude Code directly from your editor:

| Platform | Shortcut |
|----------|----------|
| Mac | `Cmd+Esc` |
| Windows/Linux | `Ctrl+Esc` |

Or click the Claude Code button in the UI.

### Diff Viewing

Code changes are displayed in the IDE's native diff viewer:
- Side-by-side comparison
- Inline highlighting
- Accept/reject controls

### Selection Context

Your current selection in the editor is automatically shared with Claude:
- Select code, then open Claude
- Claude sees your selection and can work with it

### File Reference Shortcuts

Insert file references with line ranges:

| Platform | Shortcut |
|----------|----------|
| Mac | `Cmd+Option+K` |
| Windows/Linux | `Ctrl+Alt+K` |

Creates references like: `@File.java#L1-99`

### Diagnostic Sharing

IDE diagnostics (lint errors, syntax errors, etc.) are automatically shared with Claude as you work.

---

## Part 4: Usage

### From Your IDE

1. Open your IDE's integrated terminal
2. Run `claude`
3. All integration features are now active

### From External Terminals

Connect Claude to your IDE using the `/ide` command:

```bash
# In any terminal
claude

# Connect to JetBrains IDE
> /ide
```

> **Tip:** Start Claude from the same directory as your IDE project root for best results.

---

## Part 5: Configuration

### Claude Code Settings

Configure IDE integration through Claude Code:

```bash
claude
> /config
```

Set the diff tool to `auto` for automatic IDE detection.

### Plugin Settings

Access via **Settings → Tools → Claude Code [Beta]**:

#### General Settings

| Setting | Description |
|---------|-------------|
| Claude command | Custom command to run Claude (e.g., `/usr/local/bin/claude`) |
| Suppress notification | Skip "command not found" notifications |
| Option+Enter for multi-line | (macOS) Use Option+Enter for new lines |
| Enable automatic updates | Auto-update the plugin |

#### WSL Users

Set the Claude command to:
```
wsl -d Ubuntu -- bash -lic "claude"
```

Replace `Ubuntu` with your WSL distribution name.

### ESC Key Configuration

If ESC doesn't interrupt Claude Code operations:

1. Go to **Settings → Tools → Terminal**
2. Either:
   - Uncheck "Move focus to the editor with Escape", or
   - Click "Configure terminal keybindings" and delete "Switch focus to Editor"
3. Apply changes

---

## Part 6: Remote Development

### Installing for Remote

When using JetBrains Remote Development:

> **Warning:** You must install the plugin in the remote host via **Settings → Plugin (Host)**.

The plugin must be on the remote host, not your local client.

### Verifying Remote Installation

1. Connect to your remote development environment
2. Open Settings → Plugins (Host)
3. Search for "Claude Code"
4. Install and restart if needed

---

## Part 7: WSL Configuration

### WSL Setup

WSL users may need additional configuration:

1. **Terminal configuration** - Ensure proper terminal setup
2. **Networking mode** - May need adjustments
3. **Firewall settings** - Update if IDE not detected

### WSL Command Format

In plugin settings, set Claude command:

```
wsl -d Ubuntu -- bash -lic "claude"
```

### Troubleshooting WSL

If IDE is not detected:
- Check networking mode
- Verify WSL distribution name
- Ensure Claude is installed in WSL

---

## Part 8: Troubleshooting

### Plugin Not Working

| Issue | Solution |
|-------|----------|
| Not loading | Ensure plugin is enabled in Settings |
| Wrong directory | Run Claude from project root |
| Features missing | Restart IDE completely |
| Remote not working | Install plugin on remote host |

### IDE Not Detected

1. Verify plugin is installed and enabled
2. Restart the IDE completely
3. Run Claude from integrated terminal
4. For WSL, follow WSL troubleshooting guide

### Command Not Found

If clicking Claude icon shows "command not found":

1. **Verify Claude is installed:**
   ```bash
   npm list -g @anthropic-ai/claude-code
   ```

2. **Configure command path:**
   - Go to plugin settings
   - Set full path to Claude

3. **WSL users:**
   - Use WSL command format

### Features Not Working

| Feature | Check |
|---------|-------|
| Diff viewer | Is diff tool set to `auto` in `/config`? |
| Selection context | Is plugin enabled? |
| Diagnostics | Restart IDE |

---

## Part 9: Security Considerations

### Auto-Edit Risks

With auto-edit permissions, Claude can modify IDE configuration files that may be auto-executed.

### Recommendations

1. **Use manual approval mode** for edits
2. **Use only with trusted prompts**
3. **Be aware of which files Claude can modify**
4. **Review changes before accepting**

### Safe Practices

```
DO:
- Review all proposed changes
- Use trusted repositories
- Enable manual approval mode

AVOID:
- Auto-accepting in untrusted code
- Running on sensitive configs without review
```

---

## Part 10: Advanced Usage

### Pattern 1: Quick Code Review

1. Select code in editor
2. `Cmd+Esc` / `Ctrl+Esc` to open Claude
3. "Review this code for bugs and improvements"
4. Claude sees your selection and analyzes it

### Pattern 2: Debugging with Context

1. Run code, observe error
2. IDiagnostics are automatically shared
3. Open Claude: "Fix this error"
4. Claude has access to error details

### Pattern 3: Refactoring

1. Select function to refactor
2. `Cmd+Option+K` to insert file reference
3. "Refactor this to use dependency injection"
4. Review diff in IDE's native diff viewer

### Pattern 4: Documentation Generation

1. Navigate to class/function
2. `Cmd+Option+K` for reference
3. "Generate Javadoc/docstring for this"
4. Accept changes in diff view

---

## Quick Reference

### Keyboard Shortcuts

| Action | Mac | Windows/Linux |
|--------|-----|---------------|
| Open Claude | `Cmd+Esc` | `Ctrl+Esc` |
| Insert file reference | `Cmd+Option+K` | `Ctrl+Alt+K` |

### Commands in Claude

```bash
/ide        # Connect external terminal to IDE
/config     # Configure Claude settings
```

### Plugin Settings Location

**Settings → Tools → Claude Code [Beta]**

---

## Sources

- [JetBrains IDEs - Official Documentation](https://code.claude.com/docs/en/jetbrains.md)
- [Troubleshooting Guide](https://code.claude.com/docs/en/troubleshooting.md)
- [Quickstart](https://code.claude.com/docs/en/quickstart.md)

---

## Next Steps

1. **Install the plugin** - From JetBrains Marketplace
2. **Try quick launch** - Use `Cmd/Ctrl+Esc`
3. **Test selection context** - Select code and ask about it
4. **Configure settings** - Customize for your workflow
