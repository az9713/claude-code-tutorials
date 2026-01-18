# Tutorial 28: VS Code Deep Dive

> **Master Claude Code in Visual Studio Code with native extension features**

## Overview

The VS Code extension provides a native graphical interface for Claude Code, integrated directly into your IDE. This tutorial covers installation, configuration, and advanced usage patterns.

### What You'll Learn

- Installing and configuring the VS Code extension
- Using @-mentions and inline diffs
- Commands and keyboard shortcuts
- Extension vs CLI differences
- Security considerations

### Prerequisites

- VS Code 1.98.0 or higher
- Anthropic account (Pro, Max, Team, or Enterprise)

---

## Part 1: Installation

### Installing the Extension

**Option 1: Direct Links**
- [Install for VS Code](vscode:extension/anthropic.claude-code)
- [Install for Cursor](cursor:extension/anthropic.claude-code)

**Option 2: Extensions Marketplace**
1. Press `Cmd+Shift+X` (Mac) or `Ctrl+Shift+X` (Windows/Linux)
2. Search for "Claude Code"
3. Click **Install**

**Option 3: Command Palette**
1. Press `Cmd+Shift+P` / `Ctrl+Shift+P`
2. Type "Extensions: Install Extension"
3. Search for "Claude Code"

> **Note:** You may need to restart VS Code after installation.

---

## Part 2: Getting Started

### Opening Claude Code

**Method 1: Editor Toolbar (Recommended)**
Click the Spark icon (✱) in the top-right corner of the editor.

**Method 2: Status Bar**
Click **✱ Claude Code** in the bottom-right corner.

**Method 3: Command Palette**
- `Cmd+Shift+P` / `Ctrl+Shift+P`
- Type "Claude Code"
- Select "Open in New Tab" or other options

**Method 4: Keyboard Shortcut**
- `Cmd+Esc` / `Ctrl+Esc` - Toggle focus between editor and Claude

### Your First Prompt

1. Open a file in the editor
2. Click the Spark icon
3. Ask Claude about your code:

```
What does this function do?
```

### Using @-Mentions

Reference specific files and line ranges:

**Quick mention:**
1. Select text in the editor
2. Press `Alt+K`
3. File path and line numbers are inserted into your prompt

**Manual mention:**
```
What does @src/utils/auth.ts:15-42 do?
```

---

## Part 3: Reviewing Changes

### Inline Diffs

When Claude proposes edits:

1. A diff view opens showing changes
2. Review the before/after comparison
3. Options:
   - **Accept** - Apply the changes
   - **Reject** - Discard the changes
   - **Edit** - Modify before accepting

### Auto-Accept Mode

For faster workflows, enable auto-accept:
1. Open extension settings
2. Enable auto-save and auto-accept options

> **Warning:** Use with caution. Review is recommended for important code.

---

## Part 4: Layout Customization

### Moving the Claude Panel

Drag the panel to reposition:

| Location | Description |
|----------|-------------|
| Secondary sidebar (default) | Right side of window |
| Primary sidebar | Left sidebar with Explorer, etc. |
| Editor area | As a tab alongside files |

### Terminal Mode

Prefer the CLI-style interface? Switch to terminal mode:

1. Open VS Code settings (`Cmd+,` / `Ctrl+,`)
2. Go to Extensions → Claude Code
3. Check **Use Terminal**

Or set via:
```
vscode://settings/claudeCode.useTerminal
```

---

## Part 5: Commands and Shortcuts

### VS Code Commands

| Command | Shortcut | Description |
|---------|----------|-------------|
| Focus Input | `Cmd+Esc` / `Ctrl+Esc` | Toggle focus between editor and Claude |
| Open in New Tab | `Cmd+Shift+Esc` / `Ctrl+Shift+Esc` | Open conversation as editor tab |
| New Conversation | `Cmd+N` / `Ctrl+N` | Start fresh (when Claude focused) |
| Insert @-Mention | `Alt+K` | Insert file reference with line numbers |
| Open in Side Bar | — | Open in left sidebar |
| Open in Terminal | — | Switch to terminal mode |
| Open in New Window | — | Open in separate window |
| Show Logs | — | View debug logs |
| Logout | — | Sign out of Anthropic account |

### Multiple Conversations

Use **Open in New Tab** or **Open in New Window** to run multiple conversations:
- Each tab/window has its own history
- Work on different features simultaneously
- Compare approaches side-by-side

---

## Part 6: Extension Settings

### Accessing Settings

1. `Cmd+,` / `Ctrl+,` to open VS Code settings
2. Navigate to Extensions → Claude Code

### Available Settings

| Setting | Description | Default |
|---------|-------------|---------|
| Selected Model | Default model for new conversations | default |
| Use Terminal | Launch in terminal mode | false |
| Initial Permission Mode | Approval prompts for edits | default |
| Preferred Location | Default panel location | sidebar |
| Autosave | Auto-save before Claude reads/writes | varies |
| Use Ctrl+Enter to Send | Alternative send shortcut | false |
| Respect Git Ignore | Exclude .gitignore patterns | true |
| Disable Login Prompt | Skip auth prompts (for third-party) | false |

### Claude Code Settings File

For settings shared between extension and CLI, edit `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": ["Read", "Search"],
    "deny": ["Write(~/)"]
  },
  "env": {
    "MY_VAR": "value"
  }
}
```

---

## Part 7: Third-Party Providers

### Configuring AWS Bedrock / Google Vertex AI

1. **Disable login prompt:**
   - Open settings
   - Check "Disable Login Prompt"

2. **Configure provider:**
   Edit `~/.claude/settings.json`:
   
   **For AWS Bedrock:**
   ```json
   {
     "apiProvider": "bedrock",
     "awsRegion": "us-west-2"
   }
   ```
   
   **For Google Vertex AI:**
   ```json
   {
     "apiProvider": "vertex",
     "cloudRegion": "us-east5",
     "gcpProjectId": "your-project"
   }
   ```

---

## Part 8: Extension vs CLI

### Feature Comparison

| Feature | CLI | VS Code Extension |
|---------|-----|-------------------|
| Slash commands | Full set | Subset available |
| MCP server config | Yes | Configure via CLI, use in extension |
| Checkpoints | Yes | Coming soon |
| `!` bash shortcut | Yes | No |
| Tab completion | Yes | No |

### Running CLI in VS Code

If you need CLI-only features:

1. Open integrated terminal (`` Ctrl+` `` / `` Cmd+` ``)
2. Run `claude`

The CLI automatically detects VS Code and integrates.

### Connecting External Terminal

If using an external terminal:
```bash
claude
> /ide
```

This connects Claude to VS Code for diff viewing and diagnostics.

### Switching Between Extension and CLI

Conversation history is shared:
```bash
# In terminal
claude --resume

# Opens picker to select extension conversations
```

---

## Part 9: Security Considerations

### Auto-Edit Risks

With auto-edit enabled, Claude can modify:
- VS Code `settings.json`
- `tasks.json`
- Other configuration files

These may execute automatically when VS Code processes them.

### Recommendations

1. **Enable Restricted Mode** for untrusted workspaces
2. **Use manual approval** instead of auto-accept
3. **Review changes** carefully before accepting

### Enabling Restricted Mode

1. When opening a folder, VS Code may prompt about trust
2. Choose "No, I don't trust the authors"
3. Or: File → Preferences → Workspace Trust

---

## Part 10: Troubleshooting

### Extension Won't Install

- Verify VS Code version is 1.98.0+
- Check VS Code has permission to install extensions
- Try installing from Marketplace website

### Spark Icon Not Visible

1. **Open a file** - Icon requires a file to be open
2. **Check VS Code version** - Requires 1.98.0+
3. **Restart VS Code** - Run "Developer: Reload Window"
4. **Disable conflicting extensions** - Temporarily disable AI extensions
5. **Check workspace trust** - Extension doesn't work in Restricted Mode

### Claude Not Responding

1. Check internet connection
2. Start a new conversation
3. Try the CLI for detailed error messages
4. File a bug report on GitHub

### CLI Not Connecting to IDE

- Use VS Code's integrated terminal (not external)
- Ensure `code` command is available in PATH
- Install via Command Palette: "Shell Command: Install 'code' command in PATH"

---

## Part 11: Advanced Patterns

### Pattern 1: Multi-Tab Workflow

```
Tab 1: Working on feature A
Tab 2: Debugging issue B
Tab 3: Code review for PR C
```

Each tab maintains separate context.

### Pattern 2: Selection-Based Prompting

1. Select problematic code
2. Press `Alt+K` to insert reference
3. Ask: "Why might this cause a memory leak?"

### Pattern 3: Quick Documentation

1. Select a function
2. `Alt+K`
3. "Generate JSDoc for this function"

### Pattern 4: Integrated Debugging

1. Run code, observe error
2. Copy error to Claude
3. Reference the file with `Alt+K`
4. "Fix this error in the referenced code"

---

## Quick Reference

### Keyboard Shortcuts

| Action | Mac | Windows/Linux |
|--------|-----|---------------|
| Toggle focus | `Cmd+Esc` | `Ctrl+Esc` |
| New tab | `Cmd+Shift+Esc` | `Ctrl+Shift+Esc` |
| New conversation | `Cmd+N` | `Ctrl+N` |
| Insert @-mention | `Alt+K` | `Alt+K` |

### Common Commands

```
/help          - Show available commands
/model         - Change model
/clear         - Clear conversation
/compact       - Compress context
```

### Uninstalling

1. Open Extensions (`Cmd+Shift+X` / `Ctrl+Shift+X`)
2. Search for "Claude Code"
3. Click **Uninstall**

Remove extension data:
```bash
rm -rf ~/.vscode/globalStorage/anthropic.claude-code
```

---

## Sources

- [VS Code Extension - Official Documentation](https://code.claude.com/docs/en/vs-code.md)
- [Settings Reference](https://code.claude.com/docs/en/settings.md)
- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)

---

## Next Steps

1. **Try @-mentions** - Reference code with `Alt+K`
2. **Open multiple tabs** - Work on different tasks
3. **Configure settings** - Customize for your workflow
4. **Explore CLI features** - Use integrated terminal for full capabilities
