# Tutorial 10: Claude Code IDE Integrations

## Complete Guide to Integrating Claude Code with Your Development Environment

This tutorial provides comprehensive coverage of integrating Claude Code with various IDEs and editors, enabling you to leverage AI-assisted development directly within your preferred coding environment.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [VS Code Integration](#2-vs-code-integration)
3. [JetBrains IDE Integration](#3-jetbrains-ide-integration)
4. [Terminal-Based Editors](#4-terminal-based-editors)
5. [Hand-Holding User Journey](#5-hand-holding-user-journey)
6. [Real-World Examples](#6-real-world-examples)
7. [Keyboard Shortcuts Reference](#7-keyboard-shortcuts-reference)
8. [Features Comparison](#8-features-comparison)
9. [Advanced Integration](#9-advanced-integration)
10. [Troubleshooting](#10-troubleshooting)
11. [Pros and Cons by IDE](#11-pros-and-cons-by-ide)
12. [When to Use Which](#12-when-to-use-which)
13. [Team Setup](#13-team-setup)
14. [Quick Reference](#14-quick-reference)

---

## 1. Introduction

### Claude Code as CLI vs IDE Integration

Claude Code operates primarily as a command-line interface (CLI) tool that brings AI-powered coding assistance directly to your terminal. However, modern development workflows often center around Integrated Development Environments (IDEs), making seamless integration essential for productivity.

**CLI Mode:**
```bash
# Direct terminal usage
claude "explain this codebase"
claude "fix the bug in auth.js"
claude --chat  # Interactive session
```

**IDE Integration Mode:**
- Embedded within your editor
- Context-aware suggestions
- Visual feedback and inline assistance
- No context switching required

### Benefits of IDE Integration

| Benefit | Description |
|---------|-------------|
| **Context Awareness** | IDE integrations automatically understand your current file, selection, and project structure |
| **Reduced Context Switching** | Stay in your editor instead of switching to terminal |
| **Visual Feedback** | See suggestions inline, in panels, or as overlays |
| **Keyboard-Driven** | Quick access via shortcuts without leaving your code |
| **Project Integration** | Automatic access to project files, git history, and dependencies |
| **Real-Time Assistance** | Get help as you type without manual invocation |

### Supported IDEs Overview

```
+-------------------+------------------+------------------+
|     Category      |       IDE        |  Integration     |
+-------------------+------------------+------------------+
| Microsoft         | VS Code          | Extension        |
|                   | VS Code Insiders | Extension        |
|                   | Cursor           | Native/Extension |
+-------------------+------------------+------------------+
| JetBrains         | IntelliJ IDEA    | Plugin           |
|                   | PyCharm          | Plugin           |
|                   | WebStorm         | Plugin           |
|                   | PhpStorm         | Plugin           |
|                   | RubyMine         | Plugin           |
|                   | GoLand           | Plugin           |
|                   | Rider            | Plugin           |
|                   | CLion            | Plugin           |
+-------------------+------------------+------------------+
| Terminal Editors  | Vim/Neovim       | Plugin + CLI     |
|                   | Emacs            | Package + CLI    |
+-------------------+------------------+------------------+
| Other             | Sublime Text     | Package + CLI    |
|                   | Nova             | Extension        |
+-------------------+------------------+------------------+
```

---

## 2. VS Code Integration

### Installation Steps

#### Method 1: VS Code Marketplace (Recommended)

1. **Open VS Code**
2. **Access Extensions:**
   - Press `Ctrl+Shift+X` (Windows/Linux) or `Cmd+Shift+X` (macOS)
   - Or click the Extensions icon in the Activity Bar

3. **Search and Install:**
   ```
   Search: "Claude Code"
   Publisher: Anthropic
   Click "Install"
   ```

4. **Reload VS Code** when prompted

#### Method 2: Command Line Installation

```bash
# Install via VS Code CLI
code --install-extension anthropic.claude-code

# For VS Code Insiders
code-insiders --install-extension anthropic.claude-code

# Verify installation
code --list-extensions | grep claude
```

#### Method 3: VSIX Manual Installation

```bash
# Download the VSIX file
curl -L -o claude-code.vsix https://marketplace.visualstudio.com/_apis/public/gallery/publishers/anthropic/vsextensions/claude-code/latest/vspackage

# Install manually
code --install-extension claude-code.vsix
```

### Extension Configuration

#### Settings.json Configuration

Access settings: `Ctrl+,` then click "Open Settings (JSON)"

```json
{
  // Claude Code Extension Settings
  "claudeCode.enabled": true,

  // Authentication
  "claudeCode.apiKey": "",  // Leave empty to use CLI authentication
  "claudeCode.useCliAuth": true,  // Recommended: use claude CLI auth

  // Behavior Settings
  "claudeCode.autoSuggest": true,
  "claudeCode.suggestDelay": 500,  // Milliseconds before showing suggestions
  "claudeCode.maxTokens": 4096,
  "claudeCode.temperature": 0.7,

  // UI Settings
  "claudeCode.showInlineHints": true,
  "claudeCode.panelPosition": "right",  // "right", "bottom", "left"
  "claudeCode.showStatusBar": true,

  // Context Settings
  "claudeCode.includeOpenFiles": true,
  "claudeCode.maxContextFiles": 10,
  "claudeCode.respectGitignore": true,

  // Model Selection
  "claudeCode.model": "claude-sonnet-4-20250514",

  // Performance
  "claudeCode.enableCaching": true,
  "claudeCode.streamResponses": true
}
```

#### Workspace-Specific Settings

Create `.vscode/settings.json` in your project:

```json
{
  "claudeCode.projectContext": "This is a React TypeScript project using Redux for state management",
  "claudeCode.customInstructions": "Follow our ESLint rules and use functional components",
  "claudeCode.excludePatterns": [
    "**/node_modules/**",
    "**/dist/**",
    "**/*.min.js"
  ]
}
```

### Keyboard Shortcuts

#### Default Shortcuts

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| Open Claude Panel | `Ctrl+Shift+C` | `Cmd+Shift+C` | Open the Claude Code side panel |
| Quick Ask | `Ctrl+Shift+A` | `Cmd+Shift+A` | Ask about selected code |
| Explain Code | `Ctrl+Shift+E` | `Cmd+Shift+E` | Explain selected code |
| Fix Code | `Ctrl+Shift+F` | `Cmd+Shift+F` | Fix issues in selection |
| Generate Code | `Ctrl+Shift+G` | `Cmd+Shift+G` | Generate code from comment |
| Refactor | `Ctrl+Shift+R` | `Cmd+Shift+R` | Refactor selected code |
| Add Tests | `Ctrl+Shift+T` | `Cmd+Shift+T` | Generate tests for selection |
| Toggle Inline | `Ctrl+Shift+I` | `Cmd+Shift+I` | Toggle inline suggestions |
| Accept Suggestion | `Tab` | `Tab` | Accept current suggestion |
| Dismiss Suggestion | `Esc` | `Esc` | Dismiss suggestion |
| Next Suggestion | `Alt+]` | `Option+]` | Cycle to next suggestion |
| Previous Suggestion | `Alt+[` | `Option+[` | Cycle to previous suggestion |

#### Custom Keyboard Shortcuts

Edit `keybindings.json` (`Ctrl+K Ctrl+S` then click "Open Keyboard Shortcuts (JSON)"):

```json
[
  {
    "key": "ctrl+alt+c",
    "command": "claudeCode.openChat",
    "when": "editorTextFocus"
  },
  {
    "key": "ctrl+alt+e",
    "command": "claudeCode.explainSelection",
    "when": "editorHasSelection"
  },
  {
    "key": "ctrl+alt+d",
    "command": "claudeCode.generateDocs",
    "when": "editorTextFocus"
  },
  {
    "key": "ctrl+alt+r",
    "command": "claudeCode.reviewCode",
    "when": "editorTextFocus"
  },
  {
    "key": "ctrl+enter",
    "command": "claudeCode.submitPrompt",
    "when": "claudeCode.chatFocused"
  }
]
```

### Features Available

#### 1. Side Panel Chat

The side panel provides a persistent chat interface:

```
+------------------------------------------+
|  CLAUDE CODE                         [-] |
+------------------------------------------+
|  [New Chat] [History] [Settings]         |
+------------------------------------------+
|                                          |
|  User: How do I implement               |
|  authentication in this Express app?     |
|                                          |
|  Claude: I'll help you implement         |
|  authentication. Looking at your         |
|  project structure, I recommend using    |
|  Passport.js with JWT tokens...          |
|                                          |
|  [Code block with implementation]        |
|                                          |
|  [Apply] [Copy] [Insert at Cursor]       |
|                                          |
+------------------------------------------+
|  Type your message...            [Send]  |
+------------------------------------------+
```

#### 2. Inline Suggestions

As you type, Claude provides contextual suggestions:

```javascript
// Type a comment describing what you want
// TODO: fetch user data from API and handle errors
|  // Claude suggests:
|  async function fetchUserData(userId) {
|    try {
|      const response = await fetch(`/api/users/${userId}`);
|      if (!response.ok) {
|        throw new Error(`HTTP error! status: ${response.status}`);
|      }
|      return await response.json();
|    } catch (error) {
|      console.error('Failed to fetch user:', error);
|      throw error;
|    }
|  }
|  [Tab to accept] [Esc to dismiss]
```

#### 3. Code Actions (Light Bulb Menu)

Right-click or use `Ctrl+.` on code to see Claude actions:

```
+---------------------------+
| Quick Fix...              |
| Refactor...               |
+---------------------------+
| Claude: Explain Code      |
| Claude: Fix Issues        |
| Claude: Add Documentation |
| Claude: Generate Tests    |
| Claude: Optimize          |
| Claude: Convert to...     |
+---------------------------+
```

#### 4. Problems Integration

Claude integrates with VS Code's Problems panel:

```
PROBLEMS
  src/utils.js
    ⚠ Line 15: Unused variable 'temp'  [Claude: Fix]
    ✕ Line 23: Type error              [Claude: Explain & Fix]
```

#### 5. Git Integration

Claude understands your git context:

```
SOURCE CONTROL
  Changes (3)
    M src/auth.js
    M src/routes.js
    A src/middleware.js

  [Claude: Review Changes]
  [Claude: Generate Commit Message]
  [Claude: Explain Diff]
```

### Side Panel Usage

#### Panel Sections

**1. Chat Interface**
- Persistent conversation history
- Multi-turn conversations
- Code block rendering with syntax highlighting
- One-click code application

**2. Context View**
- Currently open files
- Selected code
- Active terminal output
- Git status

**3. History**
- Previous conversations
- Saved prompts
- Bookmarked responses

**4. Quick Actions**
- Explain codebase
- Find bugs
- Generate documentation
- Run code review

#### Panel Commands

```
Available in the panel command palette (Ctrl+Shift+P):

> Claude: New Conversation
> Claude: Clear History
> Claude: Export Conversation
> Claude: Import Context
> Claude: Toggle Auto-Context
> Claude: Select Model
> Claude: View Token Usage
```

### Inline Suggestions

#### Configuration

```json
{
  "claudeCode.inlineSuggestions": {
    "enabled": true,
    "triggerCharacters": [".", "(", "{", "//", "/*"],
    "debounceMs": 300,
    "maxSuggestions": 3,
    "showConfidence": true,
    "ghostTextStyle": "italic",
    "ghostTextOpacity": 0.6
  }
}
```

#### Trigger Behaviors

| Trigger | Behavior |
|---------|----------|
| `//` followed by description | Generate code from comment |
| Empty function body | Suggest implementation |
| After `import` | Suggest imports |
| After `=` | Suggest value/expression |
| Inside string template | Suggest completion |
| After error on line | Suggest fix |

### Complete Setup Guide

#### Step-by-Step VS Code Setup

```bash
# Step 1: Ensure Claude CLI is installed and authenticated
claude --version
claude auth status

# Step 2: Install the extension
code --install-extension anthropic.claude-code

# Step 3: Open VS Code
code .

# Step 4: Verify extension is active
# Look for Claude icon in Activity Bar (left sidebar)

# Step 5: Open Command Palette and run
# > Claude: Verify Setup
```

#### First-Time Configuration Wizard

When you first activate the extension:

```
+------------------------------------------+
|     Welcome to Claude Code for VS Code   |
+------------------------------------------+
|                                          |
|  Let's get you set up!                   |
|                                          |
|  [x] Use existing CLI authentication     |
|  [ ] Enter API key manually              |
|                                          |
|  Detected CLI auth: Valid                |
|  Account: your-email@example.com         |
|                                          |
|  [Next]                                  |
+------------------------------------------+

+------------------------------------------+
|     Configure Your Preferences           |
+------------------------------------------+
|                                          |
|  Model: [claude-sonnet-4-20250514  v]    |
|                                          |
|  [x] Enable inline suggestions           |
|  [x] Show side panel on startup          |
|  [x] Include open files as context       |
|                                          |
|  [Finish Setup]                          |
+------------------------------------------+
```

---

## 3. JetBrains IDE Integration

### Supported IDEs

The Claude Code plugin supports all JetBrains IDEs built on IntelliJ Platform:

| IDE | Primary Language | Plugin Support |
|-----|------------------|----------------|
| IntelliJ IDEA | Java, Kotlin | Full |
| PyCharm | Python | Full |
| WebStorm | JavaScript, TypeScript | Full |
| PhpStorm | PHP | Full |
| RubyMine | Ruby | Full |
| GoLand | Go | Full |
| Rider | C#, .NET | Full |
| CLion | C, C++ | Full |
| DataGrip | SQL, Databases | Partial |
| Android Studio | Android/Kotlin | Full |

### Plugin Installation

#### Method 1: JetBrains Marketplace (Recommended)

1. **Open Settings/Preferences:**
   - Windows/Linux: `Ctrl+Alt+S`
   - macOS: `Cmd+,`

2. **Navigate to Plugins:**
   ```
   Settings > Plugins > Marketplace
   ```

3. **Search and Install:**
   ```
   Search: "Claude Code"
   Publisher: Anthropic
   Click "Install"
   Restart IDE when prompted
   ```

#### Method 2: Install from Disk

```bash
# Download plugin zip
curl -L -o claude-code-jetbrains.zip \
  https://plugins.jetbrains.com/files/anthropic/claude-code/latest/claude-code.zip

# In IDE: Settings > Plugins > ⚙️ > Install Plugin from Disk
# Select the downloaded zip file
```

#### Method 3: Command Line (Toolbox App)

```bash
# If using JetBrains Toolbox
# Plugin installation via CLI is not directly supported
# Use IDE GUI or web-based installation link
```

### Configuration

#### Plugin Settings

Navigate to: `Settings > Tools > Claude Code`

```
+--------------------------------------------------+
|  Claude Code Settings                            |
+--------------------------------------------------+
|                                                  |
|  Authentication                                  |
|  +-----------------------------------------+    |
|  | [x] Use CLI Authentication              |    |
|  | [ ] Use API Key                         |    |
|  |     API Key: [••••••••••••••••]         |    |
|  +-----------------------------------------+    |
|                                                  |
|  Model Configuration                             |
|  +-----------------------------------------+    |
|  | Model: [claude-sonnet-4-20250514    v]  |    |
|  | Max Tokens: [4096        ]              |    |
|  | Temperature: [0.7        ]              |    |
|  +-----------------------------------------+    |
|                                                  |
|  Behavior                                        |
|  +-----------------------------------------+    |
|  | [x] Enable inline completions           |    |
|  | [x] Show tool window on startup         |    |
|  | [x] Auto-include current file           |    |
|  | [ ] Include all open files              |    |
|  | Completion delay: [500   ] ms           |    |
|  +-----------------------------------------+    |
|                                                  |
|  [Apply]  [OK]  [Cancel]                         |
+--------------------------------------------------+
```

#### Project-Level Configuration

Create `.idea/claudeCode.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project version="4">
  <component name="ClaudeCodeSettings">
    <option name="enabled" value="true" />
    <option name="projectContext">
      Spring Boot application with PostgreSQL database
    </option>
    <option name="customInstructions">
      Follow Java 17 conventions and use constructor injection
    </option>
    <option name="excludePatterns">
      <list>
        <option value="**/target/**" />
        <option value="**/build/**" />
        <option value="**/*.class" />
      </list>
    </option>
  </component>
</project>
```

### Features

#### 1. Tool Window

Access via: `View > Tool Windows > Claude Code` or `Alt+C`

```
+--------------------------------------------------+
|  Claude Code                              [=][x] |
+--------------------------------------------------+
| [Chat] [History] [Context] [Settings]            |
+--------------------------------------------------+
|                                                  |
|  You: How do I add pagination to this            |
|  repository?                                     |
|                                                  |
|  Claude: I can see you're using Spring Data      |
|  JPA. Here's how to add pagination to your       |
|  UserRepository:                                 |
|                                                  |
|  ```java                                         |
|  public interface UserRepository extends         |
|      JpaRepository<User, Long> {                 |
|                                                  |
|      Page<User> findByActiveTrue(Pageable p);    |
|  }                                               |
|  ```                                             |
|                                                  |
|  [Apply to Editor] [Copy] [New File]             |
|                                                  |
+--------------------------------------------------+
| Message...                              [Send]   |
+--------------------------------------------------+
```

#### 2. Intention Actions

Press `Alt+Enter` on any code to see Claude actions:

```java
public class UserService {
    public User findUser(Long id) {   // Alt+Enter here
        return userRepository.findById(id).orElse(null);
    }
}

+----------------------------------+
| Claude: Explain this method      |
| Claude: Add null safety          |
| Claude: Add logging              |
| Claude: Generate JavaDoc         |
| Claude: Write unit test          |
| Claude: Suggest improvements     |
+----------------------------------+
```

#### 3. Code Completion

Claude enhances IntelliJ's code completion:

```java
// Type and wait for suggestions
public ResponseEntity<User> getUser(@PathVariable Long id) {
    // Claude suggests based on context:
    return userRepository.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}
```

#### 4. Inspections Integration

Claude adds AI-powered inspections:

```
PROBLEMS
  UserService.java
    ⚠ Potential null pointer at line 45 [Claude: Analyze]
    💡 Consider using Optional pattern [Claude: Apply Fix]
```

#### 5. Refactoring Support

Claude assists with complex refactoring:

```
Refactor Menu (Ctrl+Alt+Shift+T):
  > Extract Method...
  > Extract Variable...
  > Inline...
  > Claude: Smart Refactor...  ← AI-powered
  > Claude: Convert to Pattern...
  > Claude: Modernize Code...
```

### Keyboard Shortcuts

#### Default JetBrains Shortcuts

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| Open Tool Window | `Alt+C` | `Option+C` | Open Claude panel |
| Quick Ask | `Ctrl+Shift+C` | `Cmd+Shift+C` | Ask about selection |
| Explain Code | `Ctrl+Alt+E` | `Cmd+Option+E` | Explain selection |
| Generate Code | `Ctrl+Alt+G` | `Cmd+Option+G` | Generate from context |
| Fix Code | `Ctrl+Alt+F` | `Cmd+Option+F` | Fix selected code |
| Add Documentation | `Ctrl+Alt+D` | `Cmd+Option+D` | Generate docs |
| Run with Claude | `Ctrl+Alt+R` | `Cmd+Option+R` | Run/debug help |
| Intention Actions | `Alt+Enter` | `Option+Enter` | Show Claude actions |
| Accept Completion | `Tab` | `Tab` | Accept suggestion |
| Next Completion | `Alt+Down` | `Option+Down` | Next suggestion |
| Previous Completion | `Alt+Up` | `Option+Up` | Previous suggestion |

#### Custom Keymap Configuration

`Settings > Keymap > search "Claude"`

```xml
<!-- Add to your keymap XML for sharing -->
<keymap version="1" name="Custom with Claude">
  <action id="ClaudeCode.OpenToolWindow">
    <keyboard-shortcut first-keystroke="alt c"/>
  </action>
  <action id="ClaudeCode.QuickAsk">
    <keyboard-shortcut first-keystroke="ctrl shift c"/>
  </action>
  <action id="ClaudeCode.ExplainSelection">
    <keyboard-shortcut first-keystroke="ctrl alt e"/>
  </action>
  <action id="ClaudeCode.GenerateCode">
    <keyboard-shortcut first-keystroke="ctrl alt g"/>
  </action>
</keymap>
```

### Complete Setup Guide

#### Step-by-Step JetBrains Setup

```bash
# Step 1: Verify Claude CLI is installed
claude --version
claude auth status

# Step 2: Open your JetBrains IDE

# Step 3: Install plugin
# Settings > Plugins > Marketplace > Search "Claude Code" > Install

# Step 4: Restart IDE when prompted

# Step 5: Configure plugin
# Settings > Tools > Claude Code
# Enable "Use CLI Authentication"

# Step 6: Verify setup
# View > Tool Windows > Claude Code
# Type "Hello" and send to verify connection
```

#### First-Run Configuration

```
+--------------------------------------------------+
|  Welcome to Claude Code                          |
+--------------------------------------------------+
|                                                  |
|  Authentication Setup                            |
|                                                  |
|  We detected Claude CLI on your system.          |
|  Authentication status: ✓ Valid                  |
|                                                  |
|  [x] Use CLI authentication (recommended)        |
|  [ ] Enter API key manually                      |
|                                                  |
|  Quick Setup:                                    |
|  [x] Add Claude to Intention Actions             |
|  [x] Enable inline completions                   |
|  [x] Show Claude in Tool Windows                 |
|                                                  |
|  [Complete Setup]                                |
+--------------------------------------------------+
```

---

## 4. Terminal-Based Editors

### Vim/Neovim Integration

#### Installation Options

**Option 1: vim-plug**

```vim
" Add to ~/.vimrc or ~/.config/nvim/init.vim
call plug#begin()
Plug 'anthropic/claude-code.vim'
call plug#end()
```

**Option 2: packer.nvim (Neovim)**

```lua
-- Add to ~/.config/nvim/lua/plugins.lua
return require('packer').startup(function(use)
  use 'anthropic/claude-code.nvim'
end)
```

**Option 3: lazy.nvim (Neovim)**

```lua
-- Add to ~/.config/nvim/lua/plugins/claude.lua
return {
  'anthropic/claude-code.nvim',
  dependencies = { 'nvim-lua/plenary.nvim' },
  config = function()
    require('claude-code').setup({
      use_cli_auth = true,
      model = 'claude-sonnet-4-20250514',
    })
  end
}
```

#### Configuration

**Vim Configuration (~/.vimrc):**

```vim
" Claude Code Configuration
let g:claude_code_enabled = 1
let g:claude_code_model = 'claude-sonnet-4-20250514'
let g:claude_code_max_tokens = 4096
let g:claude_code_use_cli_auth = 1

" UI Settings
let g:claude_code_float_window = 1
let g:claude_code_float_width = 80
let g:claude_code_float_height = 20

" Key mappings
nnoremap <leader>cc :ClaudeChat<CR>
nnoremap <leader>ce :ClaudeExplain<CR>
vnoremap <leader>ce :ClaudeExplain<CR>
nnoremap <leader>cf :ClaudeFix<CR>
vnoremap <leader>cf :ClaudeFix<CR>
nnoremap <leader>cg :ClaudeGenerate<CR>
nnoremap <leader>ct :ClaudeTest<CR>
vnoremap <leader>ct :ClaudeTest<CR>
nnoremap <leader>cd :ClaudeDoc<CR>
vnoremap <leader>cd :ClaudeDoc<CR>

" Accept/dismiss suggestions
inoremap <C-y> <Cmd>ClaudeAccept<CR>
inoremap <C-n> <Cmd>ClaudeDismiss<CR>
```

**Neovim Lua Configuration (~/.config/nvim/init.lua):**

```lua
-- Claude Code setup
require('claude-code').setup({
  -- Authentication
  use_cli_auth = true,
  -- api_key = os.getenv("ANTHROPIC_API_KEY"),  -- Alternative

  -- Model settings
  model = 'claude-sonnet-4-20250514',
  max_tokens = 4096,
  temperature = 0.7,

  -- UI Configuration
  ui = {
    float = {
      enabled = true,
      width = 0.8,  -- 80% of screen
      height = 0.6,
      border = 'rounded',
    },
    split = {
      position = 'right',
      size = 60,
    },
  },

  -- Completion
  completion = {
    enabled = true,
    trigger_characters = { '.', ':', '/', '#' },
    debounce_ms = 300,
  },

  -- Keymaps (set to false to disable defaults)
  keymaps = {
    chat = '<leader>cc',
    explain = '<leader>ce',
    fix = '<leader>cf',
    generate = '<leader>cg',
    test = '<leader>ct',
    doc = '<leader>cd',
  },
})

-- Additional custom keymaps
vim.keymap.set('n', '<leader>cr', ':ClaudeRefactor<CR>', { desc = 'Claude Refactor' })
vim.keymap.set('v', '<leader>cr', ':ClaudeRefactor<CR>', { desc = 'Claude Refactor Selection' })
vim.keymap.set('n', '<leader>cp', ':ClaudePanel<CR>', { desc = 'Toggle Claude Panel' })
```

#### Vim/Neovim Commands

| Command | Description |
|---------|-------------|
| `:ClaudeChat` | Open chat window |
| `:ClaudeChat {prompt}` | Open chat with initial prompt |
| `:ClaudeExplain` | Explain current line/selection |
| `:ClaudeFix` | Fix issues in current line/selection |
| `:ClaudeGenerate` | Generate code from context |
| `:ClaudeTest` | Generate tests |
| `:ClaudeDoc` | Generate documentation |
| `:ClaudeRefactor` | Suggest refactoring |
| `:ClaudePanel` | Toggle side panel |
| `:ClaudeAccept` | Accept suggestion |
| `:ClaudeDismiss` | Dismiss suggestion |
| `:ClaudeHistory` | View conversation history |
| `:ClaudeStatus` | Check connection status |

### Emacs Integration

#### Installation

**Using use-package (Recommended):**

```elisp
;; Add to ~/.emacs.d/init.el or ~/.emacs
(use-package claude-code
  :ensure t
  :config
  (setq claude-code-use-cli-auth t)
  (setq claude-code-model "claude-sonnet-4-20250514")
  (global-claude-code-mode 1))
```

**Using straight.el:**

```elisp
(straight-use-package
 '(claude-code :type git :host github :repo "anthropic/claude-code.el"))
```

**Manual Installation:**

```bash
# Clone repository
git clone https://github.com/anthropic/claude-code.el ~/.emacs.d/lisp/claude-code

# Add to init.el
(add-to-list 'load-path "~/.emacs.d/lisp/claude-code")
(require 'claude-code)
```

#### Configuration

```elisp
;; Claude Code Configuration
(use-package claude-code
  :ensure t
  :init
  ;; Authentication
  (setq claude-code-use-cli-auth t)
  ;; (setq claude-code-api-key (getenv "ANTHROPIC_API_KEY"))  ; Alternative

  ;; Model settings
  (setq claude-code-model "claude-sonnet-4-20250514")
  (setq claude-code-max-tokens 4096)
  (setq claude-code-temperature 0.7)

  ;; UI settings
  (setq claude-code-window-position 'right)
  (setq claude-code-window-width 80)

  :config
  ;; Enable globally
  (global-claude-code-mode 1)

  ;; Key bindings
  :bind (("C-c c c" . claude-code-chat)
         ("C-c c e" . claude-code-explain)
         ("C-c c f" . claude-code-fix)
         ("C-c c g" . claude-code-generate)
         ("C-c c t" . claude-code-test)
         ("C-c c d" . claude-code-document)
         ("C-c c r" . claude-code-refactor)
         ("C-c c p" . claude-code-toggle-panel)))

;; Evil mode integration (if using Evil)
(with-eval-after-load 'evil
  (evil-define-key 'normal 'global
    (kbd "<leader>cc") 'claude-code-chat
    (kbd "<leader>ce") 'claude-code-explain
    (kbd "<leader>cf") 'claude-code-fix
    (kbd "<leader>cg") 'claude-code-generate))
```

#### Emacs Commands

| Command | Key Binding | Description |
|---------|-------------|-------------|
| `claude-code-chat` | `C-c c c` | Open chat buffer |
| `claude-code-explain` | `C-c c e` | Explain region/line |
| `claude-code-fix` | `C-c c f` | Fix code issues |
| `claude-code-generate` | `C-c c g` | Generate code |
| `claude-code-test` | `C-c c t` | Generate tests |
| `claude-code-document` | `C-c c d` | Generate docs |
| `claude-code-refactor` | `C-c c r` | Refactor code |
| `claude-code-toggle-panel` | `C-c c p` | Toggle panel |
| `claude-code-accept` | `C-c c y` | Accept suggestion |
| `claude-code-dismiss` | `C-c c n` | Dismiss suggestion |

### Terminal Multiplexer Tips

#### tmux Integration

**Recommended tmux Configuration (~/.tmux.conf):**

```bash
# Claude Code optimized tmux config

# Enable mouse support for panel interaction
set -g mouse on

# Increase scrollback for long Claude responses
set -g history-limit 50000

# Better copy mode for copying Claude's code
setw -g mode-keys vi
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-selection-and-cancel

# Quick pane for Claude CLI
bind-key C split-window -h -p 40 'claude --chat'

# Layout for development with Claude
bind-key M-c select-layout "main-vertical"  # Editor left, Claude right

# Custom layout: Editor (60%) | Claude chat (40%)
bind-key M-C run-shell "tmux split-window -h -p 40 'claude --chat' && tmux select-pane -L"

# Send selected text to Claude pane
bind-key c send-keys -t right "claude \"" \; send-keys -t right "$(tmux save-buffer -)" \; send-keys -t right "\"" Enter

# Resize panes easily
bind-key -r H resize-pane -L 5
bind-key -r J resize-pane -D 5
bind-key -r K resize-pane -U 5
bind-key -r L resize-pane -R 5
```

**Development Session Script (~/.local/bin/claude-dev):**

```bash
#!/bin/bash
# Start a development session with Claude

SESSION_NAME="${1:-dev}"
PROJECT_DIR="${2:-.}"

# Create session with Claude-ready layout
tmux new-session -d -s "$SESSION_NAME" -c "$PROJECT_DIR"

# Main editor pane (left, 60%)
tmux rename-window "code"

# Claude chat pane (right, 40%)
tmux split-window -h -p 40 -c "$PROJECT_DIR"
tmux send-keys "claude --chat" Enter

# Terminal pane (bottom right)
tmux split-window -v -p 30 -c "$PROJECT_DIR"

# Select editor pane
tmux select-pane -t 0

# Attach to session
tmux attach-session -t "$SESSION_NAME"
```

#### GNU Screen Integration

**Screen Configuration (~/.screenrc):**

```bash
# Claude Code optimized screen config

# Large scrollback
defscrollback 50000

# Status line showing pane info
hardstatus alwayslastline
hardstatus string '%{= kG}[ %{G}%H %{g}][%= %{= kw}%?%-Lw%?%{r}(%{W}%n*%f%t%?(%u)%?%{r})%{w}%?%+Lw%?%?%= %{g}][%{B} %m-%d %{W}%c %{g}]'

# Claude development layout
screen -t editor 0
screen -t claude 1 bash -c 'claude --chat'
screen -t terminal 2

# Key bindings
bind c screen -t claude bash -c 'claude --chat'
```

#### Workflow: Terminal Editor + Claude CLI

```
+----------------------------------------------------------+
|                                                          |
|   Terminal Development Workflow                          |
|                                                          |
+---------------------------+------------------------------+
|                           |                              |
|   EDITOR (vim/emacs)      |   CLAUDE CHAT                |
|                           |                              |
|   [your code here]        |   > explain the auth flow    |
|                           |                              |
|   function authenticate() |   Claude: The authentication |
|     // implementation     |   uses JWT tokens...         |
|   }                       |                              |
|                           |   > fix the bug on line 45   |
|                           |                              |
|                           |   Claude: The issue is...    |
|                           |                              |
+---------------------------+------------------------------+
|                           |                              |
|   TERMINAL                |   GIT / BUILD                |
|   npm run dev             |   git status                 |
|   Server running...       |   modified: auth.js          |
|                           |                              |
+---------------------------+------------------------------+
```

---

## 5. Hand-Holding User Journey

### Step 1: Install IDE Extension/Plugin

#### VS Code Installation Journey

```bash
# 1. Open VS Code
code

# 2. Open Extensions view
#    Press: Ctrl+Shift+X (Windows/Linux) or Cmd+Shift+X (macOS)

# 3. Search for Claude Code
#    Type: "Claude Code" in search box

# 4. Find the official extension
#    Publisher: Anthropic
#    Verified checkmark: ✓

# 5. Click Install
#    Wait for installation to complete

# 6. Reload VS Code if prompted
#    Click "Reload" button or restart VS Code

# 7. Verify installation
#    Look for Claude icon in Activity Bar (left sidebar)
#    Should see: Claude Code icon (chat bubble or 'C' logo)
```

#### JetBrains Installation Journey

```bash
# 1. Open your JetBrains IDE (IntelliJ, PyCharm, etc.)

# 2. Open Settings
#    Press: Ctrl+Alt+S (Windows/Linux) or Cmd+, (macOS)

# 3. Navigate to Plugins
#    Click: Plugins in left sidebar

# 4. Open Marketplace tab
#    Click: Marketplace (not Installed)

# 5. Search for Claude Code
#    Type: "Claude Code" in search box

# 6. Install the plugin
#    Click: Install button
#    Accept any license agreements

# 7. Restart IDE
#    Click: Restart IDE when prompted

# 8. Verify installation
#    View > Tool Windows > Claude Code should be available
```

#### Terminal Editor Installation Journey

```bash
# Neovim with lazy.nvim

# 1. Open your Neovim config
nvim ~/.config/nvim/lua/plugins/init.lua

# 2. Add the plugin specification
# Add this line:
# { 'anthropic/claude-code.nvim', dependencies = { 'nvim-lua/plenary.nvim' } }

# 3. Save and exit
# Press: :wq

# 4. Restart Neovim and sync plugins
nvim
# Run: :Lazy sync

# 5. Verify installation
# Run: :ClaudeStatus
# Should see: "Claude Code: Connected"
```

### Step 2: Configure Authentication

#### Using CLI Authentication (Recommended)

```bash
# Step 2.1: Ensure Claude CLI is authenticated
claude auth status

# Expected output:
# ✓ Authenticated as: your-email@example.com
# ✓ Organization: Your Org
# ✓ Token valid until: 2024-12-31

# If not authenticated:
claude auth login
# Follow the browser prompts to authenticate

# Step 2.2: Configure IDE to use CLI auth

# VS Code (settings.json):
{
  "claudeCode.useCliAuth": true
}

# JetBrains (Settings > Tools > Claude Code):
# Check: "Use CLI Authentication"

# Neovim (init.lua):
require('claude-code').setup({
  use_cli_auth = true,
})
```

#### Using API Key (Alternative)

```bash
# Step 2.1: Get your API key from Anthropic Console
# Visit: https://console.anthropic.com/settings/keys
# Create a new key or copy existing

# Step 2.2: Configure in IDE

# VS Code (settings.json):
{
  "claudeCode.useCliAuth": false,
  "claudeCode.apiKey": "YOUR_API_KEY"
}

# Better: Use environment variable
# In your shell profile (~/.bashrc, ~/.zshrc):
export ANTHROPIC_API_KEY="YOUR_API_KEY"

# Then in VS Code:
{
  "claudeCode.useCliAuth": false,
  "claudeCode.apiKey": "${env:ANTHROPIC_API_KEY}"
}

# JetBrains:
# Settings > Tools > Claude Code
# Uncheck "Use CLI Authentication"
# Enter API key in the field

# Neovim:
require('claude-code').setup({
  use_cli_auth = false,
  api_key = os.getenv("ANTHROPIC_API_KEY"),
})
```

### Step 3: Use Basic Features

#### VS Code: Your First Claude Interaction

```
Exercise 1: Ask a Question

1. Open any code file in VS Code

2. Open Claude Panel
   - Press: Ctrl+Shift+C (Windows/Linux) or Cmd+Shift+C (macOS)
   - Or: Click Claude icon in Activity Bar

3. Type a question:
   "What does this file do?"

4. Press Enter or click Send

5. Read Claude's response
   - Scroll through the explanation
   - Click "Copy" to copy any code blocks
   - Click "Apply" to insert code at cursor
```

```
Exercise 2: Explain Selected Code

1. Open a code file

2. Select some code
   - Click and drag to highlight a function or block

3. Ask Claude to explain
   - Press: Ctrl+Shift+E (Windows/Linux) or Cmd+Shift+E (macOS)
   - Or: Right-click > Claude: Explain Code

4. Read the explanation in the panel
```

```
Exercise 3: Fix a Bug

1. Find code with an issue (or create one)

   // Buggy code
   function divide(a, b) {
     return a / b;  // No zero check!
   }

2. Select the code

3. Ask Claude to fix
   - Press: Ctrl+Shift+F
   - Or: Type in panel: "fix this code"

4. Review suggested fix

5. Click "Apply" to insert the fix
```

#### JetBrains: Your First Claude Interaction

```
Exercise 1: Open Claude Tool Window

1. Open a project in your JetBrains IDE

2. Open Claude Tool Window
   - View > Tool Windows > Claude Code
   - Or: Press Alt+C

3. Type a question:
   "Summarize this project structure"

4. Press Enter

5. Read Claude's response
```

```
Exercise 2: Use Intention Actions

1. Open any code file

2. Place cursor on a method

3. Press Alt+Enter

4. Look for Claude options:
   - Claude: Explain this method
   - Claude: Generate tests
   - Claude: Add documentation

5. Select an action and see the result
```

### Step 4: Master Keyboard Shortcuts

#### VS Code Shortcut Practice

```
Day 1: Essential Shortcuts
- Ctrl+Shift+C : Open panel (practice 5x)
- Ctrl+Shift+E : Explain code (practice 5x)
- Tab          : Accept suggestion (practice 5x)
- Esc          : Dismiss suggestion (practice 5x)

Day 2: Action Shortcuts
- Ctrl+Shift+F : Fix code (practice 5x)
- Ctrl+Shift+G : Generate code (practice 5x)
- Ctrl+Shift+T : Generate tests (practice 5x)

Day 3: Navigation Shortcuts
- Alt+]        : Next suggestion (practice 5x)
- Alt+[        : Previous suggestion (practice 5x)
- Ctrl+Shift+I : Toggle inline (practice 5x)

Day 4: Custom Shortcuts
- Create your own shortcuts for frequent actions
- Practice new shortcuts until muscle memory forms
```

#### Shortcut Cheat Card (Print This!)

```
+--------------------------------------------------+
|          CLAUDE CODE SHORTCUTS                   |
+--------------------------------------------------+
|                                                  |
|  VS CODE                 | JETBRAINS             |
|  ------------------------|---------------------- |
|  Ctrl+Shift+C : Panel    | Alt+C : Panel         |
|  Ctrl+Shift+E : Explain  | Ctrl+Alt+E : Explain  |
|  Ctrl+Shift+F : Fix      | Ctrl+Alt+F : Fix      |
|  Ctrl+Shift+G : Generate | Ctrl+Alt+G : Generate |
|  Ctrl+Shift+T : Test     | Alt+Enter : Actions   |
|  Tab : Accept            | Tab : Accept          |
|  Esc : Dismiss           | Esc : Dismiss         |
|                                                  |
+--------------------------------------------------+
```

### Step 5: Customize for Your Workflow

#### Workflow Assessment

Answer these questions to determine your customization needs:

```
1. What type of development do you do most?
   [ ] Frontend (React, Vue, Angular)
   [ ] Backend (Node, Python, Java)
   [ ] Full-stack
   [ ] DevOps/Infrastructure
   [ ] Mobile

2. What's your editing style?
   [ ] Heavy keyboard user
   [ ] Mix of keyboard and mouse
   [ ] Primarily mouse

3. How often do you want suggestions?
   [ ] Constantly (aggressive)
   [ ] When I pause (moderate)
   [ ] Only when asked (conservative)

4. What context should Claude have?
   [ ] Only current file
   [ ] Current file + open files
   [ ] Entire project

5. What model do you prefer?
   [ ] Claude Opus (most capable)
   [ ] Claude Sonnet (balanced)
```

#### Customization Templates

**Frontend Developer Profile:**

```json
// VS Code settings.json
{
  "claudeCode.projectContext": "Frontend React TypeScript project",
  "claudeCode.customInstructions": "Use functional components, hooks, and TypeScript types. Follow Airbnb style guide.",
  "claudeCode.suggestDelay": 300,
  "claudeCode.includeOpenFiles": true,
  "claudeCode.filePatterns": ["**/*.tsx", "**/*.ts", "**/*.css"]
}
```

**Backend Developer Profile:**

```json
{
  "claudeCode.projectContext": "Backend API using Python FastAPI",
  "claudeCode.customInstructions": "Use type hints, async/await, and follow PEP 8. Include error handling.",
  "claudeCode.suggestDelay": 500,
  "claudeCode.maxContextFiles": 15,
  "claudeCode.filePatterns": ["**/*.py", "**/requirements.txt"]
}
```

**DevOps Profile:**

```json
{
  "claudeCode.projectContext": "Infrastructure as Code with Terraform and Kubernetes",
  "claudeCode.customInstructions": "Follow security best practices. Use modules and explain resource dependencies.",
  "claudeCode.suggestDelay": 400,
  "claudeCode.filePatterns": ["**/*.tf", "**/*.yaml", "**/*.yml", "**/Dockerfile"]
}
```

---

## 6. Real-World Examples

### Example 1: VS Code Full Feature Development Workflow

**Scenario:** Implement a user authentication feature in a React application.

```
Step 1: Plan the Feature
─────────────────────────

Open Claude Panel (Ctrl+Shift+C) and ask:

"I need to implement user authentication in this React app.
The app uses Redux for state and axios for API calls.
What components and files should I create?"

Claude responds with:
- Component list (LoginForm, RegisterForm, AuthProvider)
- State structure (auth reducer, actions)
- API service outline
- Protected route component

Step 2: Generate Base Components
────────────────────────────────

Create new file: src/components/Auth/LoginForm.tsx

Type comment and use Ctrl+Shift+G:
// LoginForm component with email and password fields
// Should validate inputs and dispatch login action

Claude generates:
```

```tsx
import React, { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { loginUser } from '../../store/auth/actions';
import { RootState } from '../../store/types';

interface LoginFormProps {
  onSuccess?: () => void;
}

export const LoginForm: React.FC<LoginFormProps> = ({ onSuccess }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState<Record<string, string>>({});

  const dispatch = useDispatch();
  const { loading, error } = useSelector((state: RootState) => state.auth);

  const validate = (): boolean => {
    const newErrors: Record<string, string> = {};

    if (!email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(email)) {
      newErrors.email = 'Invalid email format';
    }

    if (!password) {
      newErrors.password = 'Password is required';
    } else if (password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!validate()) return;

    try {
      await dispatch(loginUser({ email, password }));
      onSuccess?.();
    } catch (err) {
      // Error handled by reducer
    }
  };

  return (
    <form onSubmit={handleSubmit} className="login-form">
      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className={errors.email ? 'error' : ''}
        />
        {errors.email && <span className="error-text">{errors.email}</span>}
      </div>

      <div className="form-group">
        <label htmlFor="password">Password</label>
        <input
          type="password"
          id="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          className={errors.password ? 'error' : ''}
        />
        {errors.password && <span className="error-text">{errors.password}</span>}
      </div>

      {error && <div className="form-error">{error}</div>}

      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

```
Step 3: Generate Tests
──────────────────────

Select the component and press Ctrl+Shift+T:

Claude generates test file:
```

```tsx
// LoginForm.test.tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { LoginForm } from './LoginForm';
import authReducer from '../../store/auth/reducer';

const createMockStore = (initialState = {}) => {
  return configureStore({
    reducer: { auth: authReducer },
    preloadedState: { auth: { loading: false, error: null, user: null, ...initialState } },
  });
};

describe('LoginForm', () => {
  it('renders email and password fields', () => {
    render(
      <Provider store={createMockStore()}>
        <LoginForm />
      </Provider>
    );

    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
  });

  it('shows validation errors for empty fields', async () => {
    render(
      <Provider store={createMockStore()}>
        <LoginForm />
      </Provider>
    );

    fireEvent.click(screen.getByRole('button', { name: /login/i }));

    await waitFor(() => {
      expect(screen.getByText(/email is required/i)).toBeInTheDocument();
      expect(screen.getByText(/password is required/i)).toBeInTheDocument();
    });
  });

  it('shows loading state during submission', async () => {
    render(
      <Provider store={createMockStore({ loading: true })}>
        <LoginForm />
      </Provider>
    );

    expect(screen.getByRole('button', { name: /logging in/i })).toBeDisabled();
  });
});
```

```
Step 4: Debug Issues
────────────────────

When tests fail, select the error and ask:

"Why is this test failing? The mock store doesn't seem
to be providing the right state shape."

Claude identifies:
- State shape mismatch
- Provides corrected mock store setup
- Explains the fix

Step 5: Final Review
────────────────────

Select entire component and ask:

"Review this component for security issues,
accessibility, and performance problems."

Claude provides:
- Security: Add CSRF protection suggestion
- A11y: Missing aria-labels, focus management
- Performance: Memoization suggestions
```

### Example 2: JetBrains Refactoring Workflow

**Scenario:** Refactor a monolithic service class into smaller, focused services.

```
Step 1: Analyze Current Structure
─────────────────────────────────

Open Claude Tool Window (Alt+C) and ask:

"Analyze this OrderService class and suggest how to
refactor it following single responsibility principle."
```

```java
// Before: Monolithic OrderService.java (300+ lines)
@Service
public class OrderService {
    @Autowired private OrderRepository orderRepository;
    @Autowired private ProductRepository productRepository;
    @Autowired private UserRepository userRepository;
    @Autowired private EmailService emailService;
    @Autowired private PaymentGateway paymentGateway;
    @Autowired private InventoryService inventoryService;

    public Order createOrder(CreateOrderRequest request) {
        // Validate user
        User user = userRepository.findById(request.getUserId())
            .orElseThrow(() -> new UserNotFoundException(request.getUserId()));

        // Validate products and calculate total
        List<OrderItem> items = new ArrayList<>();
        BigDecimal total = BigDecimal.ZERO;
        for (OrderItemRequest itemReq : request.getItems()) {
            Product product = productRepository.findById(itemReq.getProductId())
                .orElseThrow(() -> new ProductNotFoundException(itemReq.getProductId()));
            // ... 50 more lines of validation and calculation
        }

        // Process payment
        PaymentResult result = paymentGateway.charge(user, total);
        // ... 30 more lines of payment handling

        // Update inventory
        for (OrderItem item : items) {
            inventoryService.decreaseStock(item.getProductId(), item.getQuantity());
        }

        // Send notifications
        emailService.sendOrderConfirmation(user, order);
        // ... more notification logic

        return order;
    }

    // ... 20 more methods with similar complexity
}
```

```
Claude's Analysis:
─────────────────

"This OrderService violates SRP by handling:
1. Order validation
2. Price calculation
3. Payment processing
4. Inventory management
5. Notifications

Recommended refactoring:
- OrderValidationService
- PricingService
- PaymentService (facade)
- OrderNotificationService
- OrderOrchestrator (coordinates others)"

Step 2: Generate New Services
─────────────────────────────

Use Alt+Enter on class name > Claude: Extract Service

Claude generates:
```

```java
// OrderValidationService.java
@Service
public class OrderValidationService {
    private final UserRepository userRepository;
    private final ProductRepository productRepository;

    public OrderValidationService(UserRepository userRepository,
                                   ProductRepository productRepository) {
        this.userRepository = userRepository;
        this.productRepository = productRepository;
    }

    public ValidatedOrder validate(CreateOrderRequest request) {
        User user = validateUser(request.getUserId());
        List<ValidatedItem> items = validateItems(request.getItems());
        return new ValidatedOrder(user, items);
    }

    private User validateUser(Long userId) {
        return userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
    }

    private List<ValidatedItem> validateItems(List<OrderItemRequest> items) {
        return items.stream()
            .map(this::validateItem)
            .toList();
    }

    private ValidatedItem validateItem(OrderItemRequest request) {
        Product product = productRepository.findById(request.getProductId())
            .orElseThrow(() -> new ProductNotFoundException(request.getProductId()));

        if (request.getQuantity() <= 0) {
            throw new InvalidQuantityException(request.getProductId());
        }

        return new ValidatedItem(product, request.getQuantity());
    }
}
```

```java
// PricingService.java
@Service
public class PricingService {
    private final DiscountRepository discountRepository;
    private final TaxService taxService;

    public OrderPricing calculatePricing(ValidatedOrder order, String promoCode) {
        BigDecimal subtotal = calculateSubtotal(order.getItems());
        BigDecimal discount = calculateDiscount(subtotal, promoCode);
        BigDecimal tax = taxService.calculateTax(subtotal.subtract(discount),
                                                   order.getUser().getAddress());
        BigDecimal total = subtotal.subtract(discount).add(tax);

        return new OrderPricing(subtotal, discount, tax, total);
    }

    private BigDecimal calculateSubtotal(List<ValidatedItem> items) {
        return items.stream()
            .map(item -> item.getProduct().getPrice()
                .multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    private BigDecimal calculateDiscount(BigDecimal subtotal, String promoCode) {
        if (promoCode == null) return BigDecimal.ZERO;

        return discountRepository.findByCode(promoCode)
            .map(discount -> discount.apply(subtotal))
            .orElse(BigDecimal.ZERO);
    }
}
```

```java
// OrderOrchestrator.java - Coordinates all services
@Service
public class OrderOrchestrator {
    private final OrderValidationService validationService;
    private final PricingService pricingService;
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final OrderNotificationService notificationService;
    private final OrderRepository orderRepository;

    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // 1. Validate
        ValidatedOrder validated = validationService.validate(request);

        // 2. Calculate pricing
        OrderPricing pricing = pricingService.calculatePricing(
            validated, request.getPromoCode());

        // 3. Process payment
        PaymentResult payment = paymentService.processPayment(
            validated.getUser(), pricing.getTotal());

        // 4. Create order
        Order order = createOrderEntity(validated, pricing, payment);
        order = orderRepository.save(order);

        // 5. Update inventory
        inventoryService.reserveStock(order);

        // 6. Notify (async)
        notificationService.sendOrderConfirmation(order);

        return order;
    }
}
```

```
Step 3: Generate Tests for New Services
───────────────────────────────────────

Select each new service and use Ctrl+Alt+T (JetBrains test generation)
combined with Claude to create comprehensive tests.

Step 4: Update Integration Tests
────────────────────────────────

Ask Claude:
"Update our integration tests to work with the new
service structure while maintaining the same test coverage."
```

### Example 3: Terminal Quick Bug Fix Workflow

**Scenario:** Fix a production bug using terminal editor and Claude CLI.

```bash
# Step 1: Set up split terminal
tmux split-window -h
# Left pane: vim
# Right pane: claude --chat

# Step 2: Identify the bug
# In Claude pane:
$ claude --chat
> I'm seeing this error in production:
> "TypeError: Cannot read property 'length' of undefined
>  at processItems (processor.js:45)"
> What could cause this?

Claude: This error occurs when trying to access .length on an
undefined value. In processItems at line 45, you're likely
accessing an array that wasn't initialized. Common causes:
1. Missing null check before iteration
2. API response not returning expected array
3. Race condition in data loading

Let me see the code to provide a specific fix.

# Step 3: Show the code
> Here's the processItems function:
> [paste code]

function processItems(data) {
  const items = data.response.items;
  for (let i = 0; i < items.length; i++) {  // Line 45
    // process each item
  }
}

Claude: The bug is clear. data.response.items might be undefined.
Here's the fix:

function processItems(data) {
  const items = data?.response?.items ?? [];
  if (items.length === 0) {
    console.warn('No items to process');
    return;
  }
  for (let i = 0; i < items.length; i++) {
    // process each item
  }
}

# Step 4: Apply fix in vim
# In vim pane:
:e processor.js
/processItems
# Make the edit
:w

# Step 5: Test the fix
# In terminal:
npm test -- --grep "processItems"

# Step 6: Commit
git add processor.js
git commit -m "fix: add null safety to processItems

The function was throwing TypeError when API returned
unexpected response structure. Added optional chaining
and default empty array fallback.

Fixes #1234"
```

### Example 4: Multi-IDE Project Workflow

**Scenario:** Full-stack project with separate frontend (VS Code) and backend (IntelliJ).

```
Project Structure:
─────────────────
/my-fullstack-app
  /frontend          <- VS Code (React)
  /backend           <- IntelliJ (Spring Boot)
  /shared            <- Shared types/contracts
  /infrastructure    <- Terraform (VS Code)

Workflow: Adding a New API Endpoint
───────────────────────────────────

1. DESIGN PHASE (Either IDE)

   Ask Claude:
   "I need to add a user preferences endpoint.
   Design the API contract that works for both
   React frontend and Spring Boot backend."

   Claude generates:
   - OpenAPI spec
   - TypeScript interfaces
   - Java DTOs

2. BACKEND (IntelliJ)

   Open Claude Tool Window, paste the spec:
   "Generate the Spring Boot controller, service,
   and repository for this user preferences API."

   Claude generates:
   - PreferencesController.java
   - PreferencesService.java
   - PreferencesRepository.java
   - Preference.java (entity)
   - PreferenceDTO.java

3. FRONTEND (VS Code)

   Open Claude Panel:
   "Generate the React hook and API client for
   the user preferences endpoint."

   Claude generates:
   - usePreferences.ts (custom hook)
   - preferencesApi.ts (API client)
   - types/preferences.ts (TypeScript types)

4. INTEGRATION TESTING

   In VS Code terminal:
   "Generate integration tests that verify the
   frontend correctly communicates with the
   backend for preferences."

   Claude generates:
   - Cypress tests for E2E
   - Mock server setup for frontend tests
   - TestContainers setup for backend tests

5. DEPLOYMENT (VS Code - Terraform)

   "Update the Terraform to add the new
   preferences table to RDS and update
   the API Gateway configuration."
```

### Example 5: Remote Development Workflow (SSH/Containers)

**Scenario:** Develop on a remote server or inside a container.

```
Method 1: VS Code Remote SSH
────────────────────────────

1. Install Remote-SSH extension
2. Connect to remote host
   - F1 > Remote-SSH: Connect to Host
   - Enter: user@remote-server.com

3. Claude Code works in remote context
   - Extension runs on remote
   - Has access to remote files
   - Uses remote Claude CLI if installed

Configuration for remote:
```

```json
// Remote settings (on server)
// ~/.vscode-server/data/Machine/settings.json
{
  "claudeCode.enabled": true,
  "claudeCode.useCliAuth": true,
  "claudeCode.cliPath": "/home/user/.local/bin/claude"
}
```

```
Method 2: VS Code Dev Containers
────────────────────────────────

1. Create .devcontainer/devcontainer.json:
```

```json
{
  "name": "Claude Dev Environment",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/node:1": {},
    "ghcr.io/devcontainers/features/python:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "anthropic.claude-code"
      ],
      "settings": {
        "claudeCode.enabled": true,
        "claudeCode.useCliAuth": true
      }
    }
  },
  "postCreateCommand": "npm install -g @anthropic-ai/claude-code",
  "mounts": [
    "source=${localEnv:HOME}/.config/claude,target=/home/vscode/.config/claude,type=bind"
  ]
}
```

```
2. Open in container:
   - F1 > Dev Containers: Reopen in Container

3. Claude auth is shared via mount
   - Local auth config mounted into container
   - No re-authentication needed

Method 3: JetBrains Gateway
───────────────────────────

1. Install JetBrains Gateway
2. Connect to remote IDE backend
3. Claude plugin works remotely
4. Configure remote auth:
```

```bash
# On remote server
# Install Claude CLI
npm install -g @anthropic-ai/claude-code

# Authenticate
claude auth login

# Verify
claude auth status
```

```
Method 4: Terminal with SSH + tmux
──────────────────────────────────

# Local terminal setup
ssh user@server

# On server, start persistent session
tmux new -s dev

# Set up Claude
export ANTHROPIC_API_KEY="your-key"
claude --chat  # Verify working

# Split for editor + claude
tmux split-window -h
# Left: nvim
# Right: claude --chat

# Detach and reconnect anytime
# Ctrl-b d (detach)
# ssh user@server
# tmux attach -t dev
```

---

## 7. Keyboard Shortcuts Reference

### VS Code Shortcuts

#### Default Shortcuts

| Category | Action | Windows/Linux | macOS |
|----------|--------|---------------|-------|
| **Panel** | Open Claude Panel | `Ctrl+Shift+C` | `Cmd+Shift+C` |
| | Toggle Panel | `Ctrl+Shift+P` then type "Claude Toggle" | `Cmd+Shift+P` then type "Claude Toggle" |
| | Focus Chat Input | `Ctrl+L` (when panel open) | `Cmd+L` |
| **Code Actions** | Explain Selection | `Ctrl+Shift+E` | `Cmd+Shift+E` |
| | Fix Code | `Ctrl+Shift+F` | `Cmd+Shift+F` |
| | Generate Code | `Ctrl+Shift+G` | `Cmd+Shift+G` |
| | Generate Tests | `Ctrl+Shift+T` | `Cmd+Shift+T` |
| | Add Documentation | `Ctrl+Shift+D` | `Cmd+Shift+D` |
| | Refactor | `Ctrl+Shift+R` | `Cmd+Shift+R` |
| **Suggestions** | Accept Suggestion | `Tab` | `Tab` |
| | Dismiss Suggestion | `Esc` | `Esc` |
| | Next Suggestion | `Alt+]` | `Option+]` |
| | Previous Suggestion | `Alt+[` | `Option+[` |
| | Toggle Inline | `Ctrl+Shift+I` | `Cmd+Shift+I` |
| **Chat** | Send Message | `Enter` | `Enter` |
| | New Line in Chat | `Shift+Enter` | `Shift+Enter` |
| | Clear Chat | `Ctrl+K` (in chat) | `Cmd+K` |
| | Copy Response | `Ctrl+Shift+C` (on response) | `Cmd+Shift+C` |

#### Custom Shortcut Configuration

```json
// keybindings.json
[
  // Quick access shortcuts
  {
    "key": "ctrl+.",
    "command": "claudeCode.quickActions",
    "when": "editorTextFocus"
  },
  {
    "key": "ctrl+shift+/",
    "command": "claudeCode.explainError",
    "when": "editorHasErrors"
  },

  // Panel navigation
  {
    "key": "ctrl+j ctrl+c",
    "command": "claudeCode.focusChat"
  },
  {
    "key": "ctrl+j ctrl+h",
    "command": "claudeCode.showHistory"
  },

  // Code actions with modifiers
  {
    "key": "ctrl+shift+alt+e",
    "command": "claudeCode.explainEntireFile"
  },
  {
    "key": "ctrl+shift+alt+t",
    "command": "claudeCode.generateTestFile"
  },

  // Suggestion navigation
  {
    "key": "ctrl+space",
    "command": "claudeCode.triggerSuggestion",
    "when": "editorTextFocus && !suggestWidgetVisible"
  }
]
```

### JetBrains Shortcuts

#### Default Shortcuts

| Category | Action | Windows/Linux | macOS |
|----------|--------|---------------|-------|
| **Tool Window** | Open Claude | `Alt+C` | `Option+C` |
| | Hide All Windows | `Ctrl+Shift+F12` | `Cmd+Shift+F12` |
| **Code Actions** | Explain Code | `Ctrl+Alt+E` | `Cmd+Option+E` |
| | Fix Code | `Ctrl+Alt+F` | `Cmd+Option+F` |
| | Generate Code | `Ctrl+Alt+G` | `Cmd+Option+G` |
| | Show Intentions | `Alt+Enter` | `Option+Enter` |
| **Suggestions** | Accept | `Tab` | `Tab` |
| | Dismiss | `Esc` | `Esc` |
| | Next | `Alt+Down` | `Option+Down` |
| | Previous | `Alt+Up` | `Option+Up` |
| **Navigation** | Go to Claude Action | `Ctrl+Shift+A` then "Claude" | `Cmd+Shift+A` then "Claude" |

#### Custom Keymap Configuration

```xml
<!-- Save as claude-keymap.xml and import -->
<keymap version="1" name="Default with Claude">
  <action id="ClaudeCode.OpenToolWindow">
    <keyboard-shortcut first-keystroke="alt c"/>
  </action>
  <action id="ClaudeCode.ExplainSelection">
    <keyboard-shortcut first-keystroke="ctrl alt e"/>
  </action>
  <action id="ClaudeCode.FixCode">
    <keyboard-shortcut first-keystroke="ctrl alt f"/>
  </action>
  <action id="ClaudeCode.GenerateCode">
    <keyboard-shortcut first-keystroke="ctrl alt g"/>
  </action>
  <action id="ClaudeCode.GenerateTests">
    <keyboard-shortcut first-keystroke="ctrl alt t"/>
  </action>
  <action id="ClaudeCode.ReviewCode">
    <keyboard-shortcut first-keystroke="ctrl alt r"/>
  </action>
  <action id="ClaudeCode.QuickAsk">
    <keyboard-shortcut first-keystroke="ctrl shift slash"/>
  </action>
</keymap>
```

### Terminal Editor Shortcuts

#### Vim/Neovim

| Mode | Action | Default Binding | Description |
|------|--------|-----------------|-------------|
| Normal | Chat | `<leader>cc` | Open chat window |
| Normal | Explain | `<leader>ce` | Explain current line |
| Visual | Explain | `<leader>ce` | Explain selection |
| Normal | Fix | `<leader>cf` | Fix current line |
| Visual | Fix | `<leader>cf` | Fix selection |
| Normal | Generate | `<leader>cg` | Generate code |
| Normal | Test | `<leader>ct` | Generate tests |
| Normal | Doc | `<leader>cd` | Generate documentation |
| Insert | Accept | `<C-y>` | Accept suggestion |
| Insert | Dismiss | `<C-n>` | Dismiss suggestion |
| Insert | Next | `<C-j>` | Next suggestion |
| Insert | Previous | `<C-k>` | Previous suggestion |

#### Emacs

| Action | Default Binding | Description |
|--------|-----------------|-------------|
| Chat | `C-c c c` | Open chat buffer |
| Explain | `C-c c e` | Explain region |
| Fix | `C-c c f` | Fix region |
| Generate | `C-c c g` | Generate code |
| Test | `C-c c t` | Generate tests |
| Document | `C-c c d` | Add documentation |
| Refactor | `C-c c r` | Refactor code |
| Panel | `C-c c p` | Toggle panel |
| Accept | `C-c c y` | Accept suggestion |
| Dismiss | `C-c c n` | Dismiss suggestion |

### Custom Shortcut Configuration Guide

#### Creating Muscle Memory

```
Week 1: Core Shortcuts
──────────────────────
Focus on these 5 shortcuts only:

1. Open Panel (Ctrl+Shift+C / Alt+C)
2. Explain (Ctrl+Shift+E / Ctrl+Alt+E)
3. Fix (Ctrl+Shift+F / Ctrl+Alt+F)
4. Accept (Tab)
5. Dismiss (Esc)

Practice: Use each shortcut 20+ times per day

Week 2: Generation Shortcuts
────────────────────────────
Add these 3 shortcuts:

1. Generate (Ctrl+Shift+G / Ctrl+Alt+G)
2. Test (Ctrl+Shift+T / Ctrl+Alt+T)
3. Documentation (Ctrl+Shift+D / Ctrl+Alt+D)

Practice: Use when creating new code

Week 3: Navigation Shortcuts
────────────────────────────
Add suggestion navigation:

1. Next Suggestion (Alt+] / Alt+Down)
2. Previous Suggestion (Alt+[ / Alt+Up)
3. Toggle Inline (Ctrl+Shift+I)

Week 4: Custom Shortcuts
────────────────────────
Create your own based on frequent actions
```

---

## 8. Features Comparison

### CLI vs VS Code vs JetBrains

| Feature | CLI | VS Code | JetBrains |
|---------|-----|---------|-----------|
| **Installation** | npm/pip | Extension | Plugin |
| **Learning Curve** | Medium | Low | Low |
| **Context Awareness** | Manual | Automatic | Automatic |
| **Inline Suggestions** | No | Yes | Yes |
| **Code Actions Menu** | No | Yes | Yes |
| **Side Panel** | No | Yes | Yes |
| **Multi-file Context** | Yes (explicit) | Yes (auto) | Yes (auto) |
| **Git Integration** | Yes | Yes | Yes |
| **Custom Commands** | Yes | Yes | Yes |
| **Streaming Responses** | Yes | Yes | Yes |
| **Offline Support** | No | No | No |
| **Resource Usage** | Low | Medium | Medium-High |
| **Scripting Support** | Excellent | Limited | Limited |
| **Pipeline Integration** | Excellent | Limited | Limited |

### Feature Availability Matrix

```
+---------------------------+-----+--------+-----------+
| Feature                   | CLI | VSCode | JetBrains |
+---------------------------+-----+--------+-----------+
| Chat Interface            | ✓   | ✓      | ✓         |
| Code Explanation          | ✓   | ✓      | ✓         |
| Code Generation           | ✓   | ✓      | ✓         |
| Bug Fixing                | ✓   | ✓      | ✓         |
| Test Generation           | ✓   | ✓      | ✓         |
| Documentation             | ✓   | ✓      | ✓         |
| Refactoring               | ✓   | ✓      | ✓         |
| Code Review               | ✓   | ✓      | ✓         |
+---------------------------+-----+--------+-----------+
| Inline Completions        | ✗   | ✓      | ✓         |
| Ghost Text                | ✗   | ✓      | ✓         |
| Light Bulb Actions        | ✗   | ✓      | ✓         |
| Intention Actions         | ✗   | ✓      | ✓         |
| Problems Integration      | ✗   | ✓      | ✓         |
| Quick Fixes               | ✗   | ✓      | ✓         |
+---------------------------+-----+--------+-----------+
| Batch Processing          | ✓   | ✗      | ✗         |
| Pipe Input                | ✓   | ✗      | ✗         |
| Script Automation         | ✓   | Limited| Limited   |
| CI/CD Integration         | ✓   | ✗      | ✗         |
| Pre-commit Hooks          | ✓   | ✗      | ✗         |
+---------------------------+-----+--------+-----------+
| Conversation History      | ✓   | ✓      | ✓         |
| Session Resume            | ✓   | ✓      | ✓         |
| Multiple Conversations    | ✓   | ✓      | ✓         |
| Export/Import             | ✓   | ✓      | ✓         |
+---------------------------+-----+--------+-----------+
```

### Performance Comparison

```
Startup Time (cold start):
┌────────────┬────────────┬──────────┐
│    CLI     │  VS Code   │ JetBrains│
├────────────┼────────────┼──────────┤
│   ~1s      │   ~3s      │   ~5s    │
└────────────┴────────────┴──────────┘

Memory Usage (idle):
┌────────────┬────────────┬──────────┐
│    CLI     │  VS Code   │ JetBrains│
├────────────┼────────────┼──────────┤
│  ~50MB     │  ~150MB    │  ~200MB  │
└────────────┴────────────┴──────────┘

Response Latency (typical query):
┌────────────┬────────────┬──────────┐
│    CLI     │  VS Code   │ JetBrains│
├────────────┼────────────┼──────────┤
│  ~2-5s     │  ~2-5s     │   ~2-5s  │
└────────────┴────────────┴──────────┘
(Network-bound, same for all)

Context Building:
┌────────────┬────────────┬──────────┐
│    CLI     │  VS Code   │ JetBrains│
├────────────┼────────────┼──────────┤
│  Manual    │  Auto ~1s  │  Auto ~2s│
└────────────┴────────────┴──────────┘
```

### When to Use Which

```
Decision Tree:
─────────────

                    Start
                      │
                      ▼
           ┌──────────────────┐
           │ Need automation? │
           │ (CI/CD, scripts) │
           └────────┬─────────┘
                    │
         ┌──────────┴──────────┐
         │ Yes                 │ No
         ▼                     ▼
    Use CLI         ┌──────────────────┐
                    │ Primary language?│
                    └────────┬─────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
     Java/Kotlin      JavaScript/TS      Multiple/Other
     Python           Web dev
          │                  │                  │
          ▼                  ▼                  ▼
    Use JetBrains      Use VS Code      Use VS Code
                                        (most versatile)
```

---

## 9. Advanced Integration

### Custom Commands

#### VS Code Custom Commands

Create `.vscode/claude-commands.json`:

```json
{
  "commands": [
    {
      "name": "reviewPR",
      "title": "Review Pull Request",
      "prompt": "Review this code change for:\n1. Bugs and edge cases\n2. Performance issues\n3. Security vulnerabilities\n4. Code style consistency\n\nProvide specific, actionable feedback.",
      "includeSelection": true,
      "includeDiff": true
    },
    {
      "name": "convertToTypeScript",
      "title": "Convert to TypeScript",
      "prompt": "Convert this JavaScript code to TypeScript with proper type annotations. Use strict types, avoid 'any' where possible.",
      "includeSelection": true
    },
    {
      "name": "explainArchitecture",
      "title": "Explain Architecture",
      "prompt": "Explain the architecture and design patterns used in this codebase. Include:\n1. High-level overview\n2. Key components\n3. Data flow\n4. Design patterns used",
      "includeWorkspace": true
    },
    {
      "name": "securityAudit",
      "title": "Security Audit",
      "prompt": "Perform a security audit on this code. Check for:\n1. Injection vulnerabilities\n2. Authentication/authorization issues\n3. Data exposure risks\n4. Insecure dependencies",
      "includeSelection": true,
      "includeFile": true
    }
  ]
}
```

Access via Command Palette: `> Claude: Run Custom Command`

#### JetBrains Custom Commands

Create `.idea/claude-commands.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<commands>
  <command id="springBootEndpoint" name="Generate Spring Boot Endpoint">
    <prompt><![CDATA[
Generate a complete Spring Boot REST endpoint for the following requirements:
- Follow our project's existing patterns
- Include request/response DTOs
- Add proper validation
- Include error handling
- Add Swagger documentation

Requirements:
${SELECTION}
    ]]></prompt>
    <includeSelection>true</includeSelection>
    <includeProjectContext>true</includeProjectContext>
  </command>

  <command id="jpaRepository" name="Generate JPA Repository">
    <prompt><![CDATA[
Based on this entity, generate:
1. JPA Repository interface with custom queries
2. Service class with business logic
3. Unit tests for the service

Entity:
${SELECTION}
    ]]></prompt>
    <includeSelection>true</includeSelection>
  </command>
</commands>
```

#### CLI Custom Commands

Create `~/.config/claude/commands.yaml`:

```yaml
commands:
  review:
    description: "Code review for PR"
    prompt: |
      Review this code change comprehensively:

      1. **Correctness**: Are there bugs or logic errors?
      2. **Performance**: Any performance concerns?
      3. **Security**: Security vulnerabilities?
      4. **Maintainability**: Is it readable and maintainable?
      5. **Tests**: Are tests adequate?

      Provide specific line-by-line feedback.
    includeGitDiff: true

  document:
    description: "Generate documentation"
    prompt: |
      Generate comprehensive documentation for this code:

      1. Overview/purpose
      2. API reference (if applicable)
      3. Usage examples
      4. Configuration options
      5. Error handling

  migrate:
    description: "Migration assistant"
    prompt: |
      Help migrate this code:
      - From: ${FROM_VERSION}
      - To: ${TO_VERSION}

      Identify breaking changes and provide migration steps.
```

Usage:

```bash
# Use custom command
claude --command review
claude --command document src/api/
claude --command migrate --var FROM_VERSION=React17 --var TO_VERSION=React18
```

### Workspace-Specific Settings

#### VS Code Multi-Root Workspace

Create `project.code-workspace`:

```json
{
  "folders": [
    {
      "name": "Frontend",
      "path": "./frontend"
    },
    {
      "name": "Backend",
      "path": "./backend"
    },
    {
      "name": "Shared",
      "path": "./shared"
    }
  ],
  "settings": {
    "claudeCode.enabled": true,
    "claudeCode.workspaceContext": "Full-stack application with React frontend and Node.js backend"
  },
  "claudeCode.folderSettings": {
    "Frontend": {
      "context": "React 18 with TypeScript, Redux Toolkit, and React Query",
      "customInstructions": "Use functional components and hooks. Follow Airbnb style.",
      "filePatterns": ["**/*.tsx", "**/*.ts", "**/*.css"]
    },
    "Backend": {
      "context": "Node.js Express API with TypeScript and Prisma ORM",
      "customInstructions": "Use async/await, proper error handling, follow REST conventions.",
      "filePatterns": ["**/*.ts", "**/prisma/**"]
    },
    "Shared": {
      "context": "Shared types and utilities used by both frontend and backend",
      "customInstructions": "Ensure types are compatible with both environments."
    }
  }
}
```

#### JetBrains Module-Specific Settings

For multi-module projects, create `.idea/claude-modules.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
  <component name="ClaudeCodeModuleSettings">
    <modules>
      <module name="api" context="REST API module using Spring WebFlux">
        <customInstructions>Use reactive patterns, WebFlux controllers</customInstructions>
      </module>
      <module name="domain" context="Domain model and business logic">
        <customInstructions>Follow DDD patterns, rich domain models</customInstructions>
      </module>
      <module name="infrastructure" context="Database and external service integrations">
        <customInstructions>Use repository pattern, handle external failures gracefully</customInstructions>
      </module>
    </modules>
  </component>
</project>
```

### Remote Development

#### VS Code Remote Containers + Claude

**devcontainer.json for team consistency:**

```json
{
  "name": "Team Dev Environment",
  "dockerComposeFile": "docker-compose.yml",
  "service": "dev",
  "workspaceFolder": "/workspace",

  "customizations": {
    "vscode": {
      "extensions": [
        "anthropic.claude-code",
        "ms-vscode.vscode-typescript-next",
        "dbaeumer.vscode-eslint"
      ],
      "settings": {
        "claudeCode.enabled": true,
        "claudeCode.useCliAuth": true,
        "claudeCode.model": "claude-sonnet-4-20250514",
        "claudeCode.projectContext": "E-commerce platform with microservices"
      }
    }
  },

  "mounts": [
    "source=${localEnv:HOME}/.config/claude,target=/home/node/.config/claude,type=bind"
  ],

  "postCreateCommand": "npm install -g @anthropic-ai/claude-code && claude auth status"
}
```

#### SSH Development with Persistent Claude Session

```bash
# Setup script for remote development server
#!/bin/bash

# Install Claude CLI
curl -fsSL https://claude.ai/install.sh | bash

# Create persistent tmux session
cat > ~/.local/bin/claude-workspace << 'EOF'
#!/bin/bash
SESSION="claude-dev"

tmux has-session -t $SESSION 2>/dev/null

if [ $? != 0 ]; then
  # Create new session with Claude-optimized layout
  tmux new-session -d -s $SESSION -n main

  # Main editor pane
  tmux send-keys -t $SESSION:main "nvim ." Enter

  # Claude chat pane (right side)
  tmux split-window -h -t $SESSION:main -p 35
  tmux send-keys -t $SESSION:main.1 "claude --chat" Enter

  # Terminal pane (bottom)
  tmux split-window -v -t $SESSION:main.1 -p 30

  # Select editor pane
  tmux select-pane -t $SESSION:main.0
fi

tmux attach -t $SESSION
EOF

chmod +x ~/.local/bin/claude-workspace
```

---

## 10. Troubleshooting

### Extension Not Loading

#### VS Code

```
Problem: Claude Code icon missing from Activity Bar

Solutions:

1. Check if extension is installed:
   - Ctrl+Shift+X > Installed tab
   - Search "Claude Code"
   - Should show "Enabled"

2. Check for conflicts:
   - Disable other AI extensions temporarily
   - Restart VS Code

3. Check console for errors:
   - Help > Toggle Developer Tools
   - Console tab
   - Look for "claudeCode" errors

4. Reset extension:
   - Ctrl+Shift+P > "Developer: Reload Window"
   - If persists, uninstall and reinstall

5. Check VS Code version:
   - Claude Code requires VS Code 1.80+
   - Help > About > Check version
   - Update if needed

6. Clear extension cache:
   - Close VS Code
   - Delete: ~/.vscode/extensions/.obsolete
   - Restart VS Code
```

#### JetBrains

```
Problem: Claude Code not appearing in Tools menu

Solutions:

1. Verify plugin installation:
   - Settings > Plugins > Installed
   - Search "Claude Code"
   - Ensure checkbox is checked (enabled)

2. Check for IDE compatibility:
   - Plugin requires IntelliJ Platform 2022.3+
   - Help > About > Check version

3. Invalidate caches:
   - File > Invalidate Caches / Restart
   - Select "Invalidate and Restart"

4. Check plugin conflicts:
   - Settings > Plugins
   - Disable other AI-related plugins
   - Restart and test

5. Check error log:
   - Help > Show Log in Explorer/Finder
   - Search for "ClaudeCode" in idea.log
   - Look for exception stack traces

6. Reinstall plugin:
   - Settings > Plugins > Installed
   - Find Claude Code > Uninstall
   - Restart IDE
   - Reinstall from Marketplace
```

### Authentication Issues

```
Problem: "Not authenticated" or "Invalid credentials"

Diagnosis Steps:
────────────────

1. Check CLI authentication:
   $ claude auth status

   Expected: "Authenticated as: your-email@example.com"

   If not authenticated:
   $ claude auth login

2. Check API key (if using key auth):
   $ echo $ANTHROPIC_API_KEY

   Should show your key (starts with sk-...)

3. Verify network connectivity:
   $ curl -I https://api.anthropic.com

   Should return HTTP 200 or 401 (both indicate connectivity)

4. Check IDE configuration:

   VS Code:
   - Settings > Extensions > Claude Code
   - Verify "Use CLI Auth" is checked
   - Or API key is correctly entered

   JetBrains:
   - Settings > Tools > Claude Code
   - Check authentication method
   - Test connection button

5. Token expiration:
   $ claude auth status --verbose

   Check token expiration date
   Re-authenticate if expired

6. Firewall/proxy issues:
   - Check if corporate firewall blocks api.anthropic.com
   - Configure proxy if needed:

   VS Code settings.json:
   {
     "http.proxy": "http://proxy:8080",
     "claudeCode.proxy": "http://proxy:8080"
   }
```

### Performance Problems

```
Problem: Slow responses or high resource usage

Diagnosis and Solutions:
────────────────────────

1. Check response streaming:
   - Settings > claudeCode.streamResponses: true
   - Streaming shows partial responses faster

2. Reduce context size:
   VS Code:
   {
     "claudeCode.maxContextFiles": 5,
     "claudeCode.maxContextTokens": 10000
   }

   JetBrains:
   - Settings > Tools > Claude Code > Context Settings
   - Reduce "Max files in context"

3. Exclude large files:
   {
     "claudeCode.excludePatterns": [
       "**/node_modules/**",
       "**/dist/**",
       "**/*.min.js",
       "**/package-lock.json",
       "**/*.log"
     ]
   }

4. Check memory usage:
   - VS Code: Help > Process Explorer
   - Look for "Claude" extension process
   - Should be < 200MB normally

5. Disable unused features:
   {
     "claudeCode.autoSuggest": false,  // Disable if not used
     "claudeCode.showInlineHints": false,
     "claudeCode.enableCaching": true  // Enable caching
   }

6. Network latency:
   - Check ping to api.anthropic.com
   - Consider using different model (Haiku for speed)

   {
     "claudeCode.model": "claude-haiku"  // Faster, less capable
   }
```

### Connection Issues

```
Problem: "Connection failed" or "Request timeout"

Solutions:
──────────

1. Test API connectivity:
   $ curl https://api.anthropic.com/v1/messages \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "content-type: application/json" \
       -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'

2. Check DNS resolution:
   $ nslookup api.anthropic.com

   Should resolve to valid IP addresses

3. Proxy configuration:

   Environment variables:
   $ export HTTP_PROXY=http://proxy:8080
   $ export HTTPS_PROXY=http://proxy:8080

   VS Code:
   {
     "http.proxy": "http://proxy:8080",
     "http.proxyStrictSSL": false  // If using self-signed cert
   }

4. Firewall whitelist:
   - api.anthropic.com (port 443)
   - claude.ai (port 443)

5. Timeout configuration:
   {
     "claudeCode.requestTimeout": 60000,  // 60 seconds
     "claudeCode.retryAttempts": 3
   }

6. SSL/TLS issues:
   - Check system certificates are up to date
   - For corporate environments with MITM proxy:

   {
     "claudeCode.rejectUnauthorized": false  // Use with caution
   }
```

### Platform-Specific Issues

#### Windows

```
Problem: "ENOENT" or path-related errors

Solutions:

1. Check Claude CLI installation:
   > where claude
   Should show path to claude.cmd

2. Add to PATH if missing:
   > setx PATH "%PATH%;%APPDATA%\npm"

3. Long path issues:
   - Enable long paths in Windows 10+
   - Run as admin:
   > reg add "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled /t REG_DWORD /d 1

4. Antivirus interference:
   - Add VS Code to exclusions
   - Add claude CLI to exclusions
   - Add node_modules to exclusions

5. PowerShell execution policy:
   > Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### macOS

```
Problem: "Operation not permitted" or sandbox issues

Solutions:

1. Grant full disk access:
   - System Preferences > Security & Privacy > Privacy
   - Full Disk Access
   - Add VS Code / JetBrains IDE

2. Keychain access:
   - If auth tokens stored in Keychain
   - Keychain Access app > login keychain
   - Find "claude" entries
   - Ensure app has access

3. Gatekeeper issues:
   $ xattr -d com.apple.quarantine /path/to/app

4. Node.js installation:
   - Use nvm instead of system Node
   $ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
   $ nvm install node
```

#### Linux

```
Problem: "EACCES" permission errors

Solutions:

1. Fix npm permissions:
   $ mkdir -p ~/.npm-global
   $ npm config set prefix '~/.npm-global'
   $ echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
   $ source ~/.bashrc

2. SELinux issues (RHEL/CentOS):
   $ sudo setsebool -P httpd_can_network_connect 1

3. AppArmor issues (Ubuntu):
   $ sudo aa-complain /usr/bin/code

4. Missing dependencies:
   $ sudo apt install libsecret-1-0  # For credential storage

5. Font rendering (for UI):
   $ sudo apt install fonts-noto
```

---

## 11. Pros and Cons by IDE

### CLI Advantages and Disadvantages

#### Advantages

```
+ Lightweight
  - Minimal resource usage (~50MB RAM)
  - Fast startup (~1 second)
  - No GUI overhead

+ Automation-Friendly
  - Easily scripted
  - Pipeline integration
  - CI/CD compatible
  - Pre-commit hooks

+ Universal Access
  - Works over SSH
  - Works in containers
  - Works on any terminal
  - No X11/display needed

+ Full Control
  - Precise file selection
  - Explicit context management
  - Detailed output control
  - JSON output for parsing

+ Version Control Integration
  - Git-aware operations
  - Diff analysis
  - Commit message generation

+ Flexibility
  - Combine with any editor
  - Custom workflows
  - Shell integration
```

#### Disadvantages

```
- No Real-Time Suggestions
  - Must explicitly invoke
  - No inline completions
  - No ghost text

- Manual Context Management
  - Must specify files explicitly
  - No automatic project awareness
  - More typing required

- Limited Visual Feedback
  - Text-only responses
  - No code highlighting in prompt
  - No side-by-side comparison

- Steeper Learning Curve
  - Must learn CLI syntax
  - Remember flags and options
  - No discoverable UI

- Context Switching
  - Must switch to terminal
  - Copy/paste code manually
  - No direct code insertion
```

### VS Code Advantages and Disadvantages

#### Advantages

```
+ Seamless Integration
  - Native look and feel
  - Keyboard-centric workflow
  - Quick actions available

+ Inline Suggestions
  - Ghost text completions
  - Tab to accept
  - Context-aware suggestions

+ Visual Interface
  - Side panel for chat
  - Code highlighting
  - Diff views

+ Automatic Context
  - Open files included
  - Selection awareness
  - Project structure understanding

+ Ecosystem Integration
  - Works with other extensions
  - Shares settings sync
  - Consistent across machines

+ Low Learning Curve
  - Familiar VS Code patterns
  - Discoverable features
  - Good documentation

+ Extension Ecosystem
  - Combines with Git extensions
  - Works with language servers
  - Enhances existing workflow
```

#### Disadvantages

```
- Resource Usage
  - Higher RAM (~150MB for extension)
  - CPU for background indexing
  - Slower on large projects

- VS Code Dependency
  - Must use VS Code
  - Not available standalone
  - Tied to VS Code updates

- Limited Scripting
  - Hard to automate
  - No pipeline integration
  - Manual operation only

- Potential Conflicts
  - Other AI extensions
  - Keyboard shortcut conflicts
  - Settings conflicts

- Network Dependency
  - Requires constant connection
  - No offline mode
  - Latency affects UX
```

### JetBrains Advantages and Disadvantages

#### Advantages

```
+ Deep IDE Integration
  - Works with refactoring tools
  - Integrates with inspections
  - Uses IDE's code understanding

+ Language-Specific Intelligence
  - Java/Kotlin expertise
  - Framework awareness (Spring, etc.)
  - Library knowledge

+ Professional Features
  - Database integration
  - Debugger awareness
  - Build tool integration

+ Enterprise Ready
  - Works with corporate proxies
  - LDAP/SSO compatible
  - Audit logging

+ Powerful Intentions
  - Alt+Enter integration
  - Context-appropriate actions
  - Smart suggestions

+ Project Model Awareness
  - Module understanding
  - Dependency awareness
  - Build configuration knowledge
```

#### Disadvantages

```
- Highest Resource Usage
  - Heavy IDE + plugin
  - ~200MB+ for Claude
  - Slower startup

- Cost
  - JetBrains IDEs are paid
  - Additional plugin may have costs
  - Enterprise licensing complex

- Learning Curve
  - JetBrains-specific patterns
  - Different from other IDEs
  - Complex settings

- Platform Limitations
  - Some features IDE-specific
  - Updates tied to IDE cycle
  - Plugin compatibility issues

- Overkill for Simple Tasks
  - Heavy for quick edits
  - Slow for small projects
  - Resource intensive
```

### Decision Guide

```
Choose CLI if:
─────────────
□ You work primarily in terminal
□ You need automation/scripting
□ You use multiple editors
□ You work over SSH frequently
□ You want minimal resource usage
□ You need CI/CD integration

Choose VS Code if:
──────────────────
□ You're a web developer
□ You work with JavaScript/TypeScript
□ You want inline suggestions
□ You prefer visual feedback
□ You want low resource usage
□ You're starting with AI coding tools

Choose JetBrains if:
────────────────────
□ You work with Java/Kotlin
□ You need deep IDE integration
□ You work on large enterprise projects
□ You want framework-specific help
□ You already use JetBrains IDEs
□ You need advanced refactoring
```

---

## 12. When to Use Which

### CLI Best For

```
1. AUTOMATION AND SCRIPTING

   # Automated code review in CI
   claude --json "review this PR for security issues" \
     < git_diff.txt \
     | jq '.issues'

   # Batch documentation generation
   for file in src/*.ts; do
     claude "generate JSDoc for this file" < "$file" > "${file%.ts}.md"
   done

   # Pre-commit hook
   #!/bin/bash
   staged=$(git diff --cached)
   claude "check this code for issues" <<< "$staged"

2. REMOTE/SSH DEVELOPMENT

   # Quick fix on production server
   ssh prod-server
   claude "explain why this process is using 100% CPU" < /proc/$(pgrep myapp)/status

   # Log analysis
   claude "summarize errors in this log" < /var/log/app.log

3. PIPELINE INTEGRATION

   # GitHub Actions step
   - name: AI Code Review
     run: |
       claude "review changes" < <(git diff ${{ github.event.before }} ${{ github.sha }})

4. QUICK QUERIES

   # One-off questions
   claude "what's the difference between let and const?"

   # Command help
   claude "explain this bash command: find . -name '*.js' -exec grep -l 'TODO' {} +"

5. DATA PROCESSING

   # Analyze structured data
   cat data.json | claude "summarize this data and identify anomalies"

   # Convert formats
   claude "convert this XML to JSON" < config.xml > config.json
```

### VS Code Best For

```
1. DAY-TO-DAY CODING

   - Writing new features
   - Fixing bugs with context
   - Exploring unfamiliar code
   - Code completion assistance

2. WEB DEVELOPMENT

   - React/Vue/Angular development
   - CSS/styling help
   - JavaScript/TypeScript projects
   - HTML markup generation

3. LEARNING AND EXPLORATION

   - Understanding new codebases
   - Learning new frameworks
   - Getting explanations inline
   - Discovering patterns

4. DOCUMENTATION

   - Writing README files
   - Generating API docs
   - Adding code comments
   - Creating examples

5. RAPID PROTOTYPING

   - Quick component generation
   - API client scaffolding
   - Test file creation
   - Configuration setup
```

### JetBrains Best For

```
1. ENTERPRISE JAVA DEVELOPMENT

   - Spring Boot applications
   - Complex refactoring
   - Design pattern implementation
   - Framework integration

2. KOTLIN/ANDROID DEVELOPMENT

   - Android app development
   - Kotlin idiom usage
   - Gradle configuration
   - Android-specific patterns

3. DATABASE WORK (DataGrip)

   - SQL query optimization
   - Schema design
   - Migration scripts
   - Query explanation

4. LARGE-SCALE REFACTORING

   - Cross-module changes
   - Type hierarchy modifications
   - Rename across project
   - Extract method/class

5. DEBUGGING ASSISTANCE

   - Exception analysis
   - Stack trace explanation
   - Memory leak investigation
   - Performance profiling help
```

### Hybrid Approaches

```
WORKFLOW 1: VS Code + CLI

Daily development in VS Code:
- Write code with inline suggestions
- Get explanations via panel
- Generate tests with shortcuts

Use CLI for:
- Commit message generation
- Pre-push code review
- CI/CD integration

Example:
# In VS Code: develop feature
# Then in terminal:
git add -A
claude "generate commit message" < <(git diff --cached) | git commit -F -


WORKFLOW 2: JetBrains + CLI

Core development in JetBrains:
- Java/Kotlin coding
- Refactoring
- Debugging

Use CLI for:
- Build script help
- DevOps tasks
- Documentation generation

Example:
# In IntelliJ: write Java code
# In terminal:
claude "generate README for this project" --include src/main/java > README.md


WORKFLOW 3: Multiple IDEs

Frontend (VS Code):
- React components
- CSS styling
- TypeScript

Backend (IntelliJ):
- Java services
- Database queries
- API design

CLI (both):
- Git operations
- Deployment scripts
- Integration testing


WORKFLOW 4: Terminal Editor + CLI

Vim/Neovim for editing:
- Direct file editing
- Multi-file changes
- Vim motions

tmux pane with Claude CLI:
- Continuous conversation
- Quick queries
- Context preserved

Example layout:
┌─────────────────┬────────────────┐
│                 │                │
│   VIM EDITOR    │  CLAUDE CHAT   │
│                 │                │
│   [code here]   │  > explain     │
│                 │    this func   │
│                 │                │
├─────────────────┴────────────────┤
│           TERMINAL               │
│  $ npm test                      │
└──────────────────────────────────┘
```

---

## 13. Team Setup

### Standardizing IDE Configuration

#### VS Code Team Setup

**Create `.vscode/extensions.json`:**

```json
{
  "recommendations": [
    "anthropic.claude-code",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next"
  ],
  "unwantedRecommendations": [
    "other-ai-extension.competitor"
  ]
}
```

**Create `.vscode/settings.json` (committed to repo):**

```json
{
  // Claude Code - Team Settings
  "claudeCode.enabled": true,
  "claudeCode.useCliAuth": true,
  "claudeCode.model": "claude-sonnet-4-20250514",

  // Project context
  "claudeCode.projectContext": "E-commerce platform using React, Node.js, and PostgreSQL",
  "claudeCode.customInstructions": "Follow team coding standards in CONTRIBUTING.md. Use TypeScript strict mode. Write tests for all new code.",

  // Consistent behavior
  "claudeCode.maxContextFiles": 10,
  "claudeCode.excludePatterns": [
    "**/node_modules/**",
    "**/dist/**",
    "**/coverage/**",
    "**/*.min.js",
    "**/package-lock.json"
  ],

  // Code style enforcement
  "claudeCode.codeStyle": {
    "indentation": "spaces",
    "indentSize": 2,
    "semicolons": true,
    "quotes": "single"
  }
}
```

#### JetBrains Team Setup

**Create `.idea/claudeCode.xml`:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project version="4">
  <component name="ClaudeCodeProjectSettings">
    <option name="enabled" value="true" />
    <option name="useCliAuth" value="true" />
    <option name="model" value="claude-sonnet-4-20250514" />

    <option name="projectContext">
      Spring Boot microservices application with PostgreSQL and Redis.
      Uses Maven for builds and follows clean architecture.
    </option>

    <option name="customInstructions">
      Follow Google Java Style Guide.
      Use constructor injection for dependencies.
      All public methods must have JavaDoc.
      Follow TDD approach.
    </option>

    <excludePatterns>
      <pattern>**/target/**</pattern>
      <pattern>**/build/**</pattern>
      <pattern>**/*.class</pattern>
      <pattern>**/generated/**</pattern>
    </excludePatterns>
  </component>
</project>
```

**Add to `.gitignore`:**

```gitignore
# Keep Claude settings but ignore personal preferences
!.idea/claudeCode.xml
.idea/claudeCode.local.xml
```

### Shared Settings

#### Team Configuration File

**Create `.claude/team-config.yaml` (committed to repo):**

```yaml
# Team Claude Code Configuration
version: 1

# Project context shared across all team members
project:
  name: "MyProject"
  description: |
    E-commerce platform with microservices architecture.
    Frontend: React 18 with TypeScript
    Backend: Node.js with Express
    Database: PostgreSQL with Prisma ORM

  techStack:
    - React 18
    - TypeScript 5
    - Node.js 20
    - Express 4
    - PostgreSQL 15
    - Prisma ORM
    - Jest for testing
    - Docker for deployment

# Coding standards
standards:
  style:
    indentation: "2 spaces"
    semicolons: true
    quotes: "single"
    maxLineLength: 100

  documentation:
    required: true
    format: "JSDoc"

  testing:
    required: true
    minCoverage: 80
    framework: "Jest"

# Custom instructions for all team members
instructions: |
  When generating code:
  1. Follow our ESLint configuration
  2. Write unit tests for all new functions
  3. Use TypeScript strict mode
  4. Add JSDoc comments for public APIs
  5. Handle errors appropriately
  6. Use async/await over callbacks

  When explaining code:
  1. Be concise but thorough
  2. Highlight potential issues
  3. Suggest improvements

  When reviewing code:
  1. Check for security vulnerabilities
  2. Verify error handling
  3. Ensure test coverage
  4. Look for performance issues

# Custom commands for the team
commands:
  review:
    description: "Standard code review"
    prompt: |
      Review this code following our team standards:
      - Check against ESLint rules
      - Verify TypeScript types
      - Ensure proper error handling
      - Check for security issues
      - Verify test coverage

  feature:
    description: "Generate feature scaffold"
    prompt: |
      Generate a complete feature including:
      - React component with TypeScript
      - Unit tests with Jest
      - Storybook story
      - API integration if needed
```

#### Import Configuration in IDEs

**VS Code `settings.json`:**

```json
{
  "claudeCode.teamConfigPath": "${workspaceFolder}/.claude/team-config.yaml"
}
```

**JetBrains (in claudeCode.xml):**

```xml
<option name="teamConfigPath" value="$PROJECT_DIR$/.claude/team-config.yaml" />
```

### Onboarding New Developers

#### Onboarding Checklist

```markdown
# Claude Code Onboarding Checklist

## Day 1: Installation

- [ ] Install Claude CLI
  ```bash
  npm install -g @anthropic-ai/claude-code
  ```

- [ ] Authenticate with Claude
  ```bash
  claude auth login
  ```

- [ ] Verify authentication
  ```bash
  claude auth status
  ```

- [ ] Install IDE extension/plugin
  - VS Code: Search "Claude Code" in Extensions
  - JetBrains: Search "Claude Code" in Plugins

## Day 1: Configuration

- [ ] Clone team repository
  ```bash
  git clone <repo-url>
  ```

- [ ] Open in your IDE
  - Team settings will load automatically
  - Verify Claude icon appears

- [ ] Test the connection
  - Open Claude panel
  - Type "Hello" and verify response

## Day 2: Learn Basic Shortcuts

- [ ] Practice these shortcuts 10 times each:
  - Open panel: Ctrl+Shift+C (VS Code) / Alt+C (JetBrains)
  - Explain code: Ctrl+Shift+E / Ctrl+Alt+E
  - Accept suggestion: Tab
  - Dismiss suggestion: Esc

## Day 2-3: Hands-On Practice

- [ ] Complete these exercises:

  Exercise 1: Ask Claude to explain a complex function
  - Find a function you don't understand
  - Select it and press the Explain shortcut
  - Read the explanation

  Exercise 2: Generate a test
  - Select a function without tests
  - Ask Claude to generate tests
  - Review and refine the tests

  Exercise 3: Fix a bug
  - Find a TODO or known issue
  - Select the code and ask Claude to fix it
  - Review the suggestion before applying

## Day 4-5: Advanced Features

- [ ] Learn custom team commands
  - Review .claude/team-config.yaml
  - Try each custom command

- [ ] Understand context management
  - See how files are included
  - Learn to exclude irrelevant files

## Week 2: Integration into Workflow

- [ ] Use Claude for your first feature
- [ ] Use Claude for your first code review
- [ ] Share feedback with team
```

#### Onboarding Script

```bash
#!/bin/bash
# save as: scripts/setup-claude.sh

echo "🚀 Claude Code Setup for New Team Members"
echo "=========================================="

# Check prerequisites
echo ""
echo "Checking prerequisites..."

if ! command -v node &> /dev/null; then
    echo "❌ Node.js is not installed. Please install Node.js 18+ first."
    exit 1
fi

if ! command -v code &> /dev/null && ! command -v idea &> /dev/null; then
    echo "⚠️  No supported IDE detected. Please install VS Code or JetBrains IDE."
fi

# Install Claude CLI
echo ""
echo "Installing Claude CLI..."
npm install -g @anthropic-ai/claude-code

# Verify installation
echo ""
echo "Verifying installation..."
claude --version

# Authenticate
echo ""
echo "Please authenticate with Claude..."
claude auth login

# Verify auth
echo ""
echo "Verifying authentication..."
claude auth status

# Install VS Code extension (if VS Code is available)
if command -v code &> /dev/null; then
    echo ""
    echo "Installing VS Code extension..."
    code --install-extension anthropic.claude-code
fi

# Final verification
echo ""
echo "=========================================="
echo "✅ Setup complete!"
echo ""
echo "Next steps:"
echo "1. Open your IDE and verify the Claude icon appears"
echo "2. Read the team config at .claude/team-config.yaml"
echo "3. Try the basic shortcuts from the onboarding checklist"
echo "4. Ask in #dev-help if you have questions"
echo ""
```

---

## 14. Quick Reference

### Installation Commands

```bash
# ===== CLAUDE CLI =====

# npm (recommended)
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version

# Authenticate
claude auth login

# Check auth status
claude auth status

# ===== VS CODE EXTENSION =====

# Command line
code --install-extension anthropic.claude-code

# Or via marketplace:
# 1. Ctrl+Shift+X
# 2. Search "Claude Code"
# 3. Click Install

# ===== JETBRAINS PLUGIN =====

# Via IDE:
# 1. Settings > Plugins > Marketplace
# 2. Search "Claude Code"
# 3. Click Install > Restart IDE

# ===== VIM/NEOVIM =====

# vim-plug
# Add to .vimrc: Plug 'anthropic/claude-code.vim'
# Run: :PlugInstall

# lazy.nvim
# Add to plugins: { 'anthropic/claude-code.nvim' }
# Run: :Lazy sync

# ===== EMACS =====

# use-package
# Add to init.el: (use-package claude-code :ensure t)
# Run: M-x package-refresh-contents, then M-x package-install claude-code
```

### Shortcut Cheat Sheets

#### VS Code Cheat Sheet

```
╔═══════════════════════════════════════════════════════════╗
║              VS CODE CLAUDE CODE SHORTCUTS                 ║
╠═══════════════════════════════════════════════════════════╣
║                                                           ║
║  PANEL OPERATIONS                                         ║
║  ─────────────────                                        ║
║  Ctrl+Shift+C      Open Claude Panel                      ║
║  Ctrl+L            Focus chat input (when panel open)     ║
║  Enter             Send message                           ║
║  Shift+Enter       New line in chat                       ║
║                                                           ║
║  CODE ACTIONS                                             ║
║  ─────────────                                            ║
║  Ctrl+Shift+E      Explain selected code                  ║
║  Ctrl+Shift+F      Fix issues in selection                ║
║  Ctrl+Shift+G      Generate code                          ║
║  Ctrl+Shift+T      Generate tests                         ║
║  Ctrl+Shift+D      Add documentation                      ║
║  Ctrl+Shift+R      Refactor selection                     ║
║                                                           ║
║  SUGGESTIONS                                              ║
║  ───────────                                              ║
║  Tab               Accept suggestion                      ║
║  Esc               Dismiss suggestion                     ║
║  Alt+]             Next suggestion                        ║
║  Alt+[             Previous suggestion                    ║
║  Ctrl+Shift+I      Toggle inline suggestions              ║
║                                                           ║
║  macOS: Replace Ctrl with Cmd                             ║
║                                                           ║
╚═══════════════════════════════════════════════════════════╝
```

#### JetBrains Cheat Sheet

```
╔═══════════════════════════════════════════════════════════╗
║            JETBRAINS CLAUDE CODE SHORTCUTS                 ║
╠═══════════════════════════════════════════════════════════╣
║                                                           ║
║  TOOL WINDOW                                              ║
║  ───────────                                              ║
║  Alt+C             Open Claude Tool Window                ║
║  Escape            Return focus to editor                 ║
║                                                           ║
║  CODE ACTIONS                                             ║
║  ─────────────                                            ║
║  Ctrl+Alt+E        Explain selected code                  ║
║  Ctrl+Alt+F        Fix issues in selection                ║
║  Ctrl+Alt+G        Generate code                          ║
║  Ctrl+Alt+T        Generate tests (also IDE shortcut)     ║
║  Ctrl+Alt+D        Add documentation                      ║
║  Alt+Enter         Show intention actions (incl. Claude)  ║
║                                                           ║
║  SUGGESTIONS                                              ║
║  ───────────                                              ║
║  Tab               Accept suggestion                      ║
║  Escape            Dismiss suggestion                     ║
║  Alt+Down          Next suggestion                        ║
║  Alt+Up            Previous suggestion                    ║
║                                                           ║
║  NAVIGATION                                               ║
║  ──────────                                               ║
║  Ctrl+Shift+A      Find Action > type "Claude"            ║
║  Double Shift      Search Everywhere > "Claude"           ║
║                                                           ║
║  macOS: Replace Ctrl with Cmd, Alt with Option            ║
║                                                           ║
╚═══════════════════════════════════════════════════════════╝
```

#### Terminal Cheat Sheet

```
╔═══════════════════════════════════════════════════════════╗
║              TERMINAL CLAUDE CODE SHORTCUTS                ║
╠═══════════════════════════════════════════════════════════╣
║                                                           ║
║  CLI COMMANDS                                             ║
║  ─────────────                                            ║
║  claude "prompt"           One-shot query                 ║
║  claude --chat             Interactive chat mode          ║
║  claude -f file.js         Include file in context        ║
║  claude --json             Output as JSON                 ║
║  claude | command          Pipe output                    ║
║  command | claude          Pipe input                     ║
║                                                           ║
║  VIM/NEOVIM (with plugin)                                 ║
║  ─────────────────────────                                ║
║  <leader>cc        Open chat window                       ║
║  <leader>ce        Explain line/selection                 ║
║  <leader>cf        Fix line/selection                     ║
║  <leader>cg        Generate code                          ║
║  <leader>ct        Generate tests                         ║
║  <leader>cd        Generate documentation                 ║
║  Ctrl+y (insert)   Accept suggestion                      ║
║  Ctrl+n (insert)   Dismiss suggestion                     ║
║                                                           ║
║  EMACS (with package)                                     ║
║  ────────────────────                                     ║
║  C-c c c           Open chat buffer                       ║
║  C-c c e           Explain region                         ║
║  C-c c f           Fix region                             ║
║  C-c c g           Generate code                          ║
║  C-c c t           Generate tests                         ║
║  C-c c d           Generate documentation                 ║
║                                                           ║
║  TMUX INTEGRATION                                         ║
║  ────────────────                                         ║
║  Prefix + C        Open Claude pane (custom binding)      ║
║  Prefix + [        Enter copy mode (copy Claude output)   ║
║                                                           ║
╚═══════════════════════════════════════════════════════════╝
```

### Configuration Templates

#### VS Code Template

```json
// .vscode/settings.json - Complete template
{
  // ===== Claude Code Settings =====

  // Authentication (prefer CLI auth)
  "claudeCode.enabled": true,
  "claudeCode.useCliAuth": true,

  // Model configuration
  "claudeCode.model": "claude-sonnet-4-20250514",
  "claudeCode.maxTokens": 4096,
  "claudeCode.temperature": 0.7,

  // UI preferences
  "claudeCode.showInlineHints": true,
  "claudeCode.panelPosition": "right",
  "claudeCode.showStatusBar": true,
  "claudeCode.streamResponses": true,

  // Context management
  "claudeCode.includeOpenFiles": true,
  "claudeCode.maxContextFiles": 10,
  "claudeCode.respectGitignore": true,
  "claudeCode.excludePatterns": [
    "**/node_modules/**",
    "**/dist/**",
    "**/build/**",
    "**/*.min.js",
    "**/package-lock.json",
    "**/.git/**"
  ],

  // Inline suggestions
  "claudeCode.autoSuggest": true,
  "claudeCode.suggestDelay": 500,

  // Project-specific context
  "claudeCode.projectContext": "Describe your project here",
  "claudeCode.customInstructions": "Add any specific coding standards or preferences",

  // Performance
  "claudeCode.enableCaching": true,
  "claudeCode.requestTimeout": 30000
}
```

#### JetBrains Template

```xml
<!-- .idea/claudeCode.xml - Complete template -->
<?xml version="1.0" encoding="UTF-8"?>
<project version="4">
  <component name="ClaudeCodeSettings">
    <!-- Authentication -->
    <option name="enabled" value="true" />
    <option name="useCliAuth" value="true" />

    <!-- Model Configuration -->
    <option name="model" value="claude-sonnet-4-20250514" />
    <option name="maxTokens" value="4096" />
    <option name="temperature" value="0.7" />

    <!-- UI Preferences -->
    <option name="showToolWindowOnStartup" value="false" />
    <option name="streamResponses" value="true" />

    <!-- Context Management -->
    <option name="includeCurrentFile" value="true" />
    <option name="includeOpenFiles" value="false" />
    <option name="maxContextFiles" value="10" />

    <!-- Exclude Patterns -->
    <excludePatterns>
      <pattern>**/target/**</pattern>
      <pattern>**/build/**</pattern>
      <pattern>**/*.class</pattern>
      <pattern>**/node_modules/**</pattern>
      <pattern>**/.gradle/**</pattern>
    </excludePatterns>

    <!-- Project Context -->
    <option name="projectContext">
      Describe your project here.
      Include technology stack, architecture, and conventions.
    </option>

    <!-- Custom Instructions -->
    <option name="customInstructions">
      Add any specific coding standards or preferences.
      - Follow style guide X
      - Use pattern Y
      - Avoid approach Z
    </option>

    <!-- Inline Suggestions -->
    <option name="enableInlineCompletions" value="true" />
    <option name="completionDelay" value="500" />
  </component>
</project>
```

#### Neovim Template

```lua
-- ~/.config/nvim/lua/plugins/claude.lua - Complete template
return {
  'anthropic/claude-code.nvim',
  dependencies = { 'nvim-lua/plenary.nvim' },
  config = function()
    require('claude-code').setup({
      -- Authentication
      use_cli_auth = true,
      -- api_key = os.getenv("ANTHROPIC_API_KEY"),  -- Alternative

      -- Model Configuration
      model = 'claude-sonnet-4-20250514',
      max_tokens = 4096,
      temperature = 0.7,

      -- UI Configuration
      ui = {
        float = {
          enabled = true,
          width = 0.8,
          height = 0.6,
          border = 'rounded',
        },
        split = {
          position = 'right',
          size = 60,
        },
      },

      -- Completion Settings
      completion = {
        enabled = true,
        trigger_characters = { '.', ':', '/', '#' },
        debounce_ms = 300,
      },

      -- Context Settings
      context = {
        include_current_file = true,
        include_open_buffers = false,
        max_files = 10,
        exclude_patterns = {
          'node_modules/',
          'dist/',
          'build/',
          '.git/',
        },
      },

      -- Project Context
      project_context = [[
        Describe your project here.
      ]],

      custom_instructions = [[
        Add any specific coding standards.
      ]],

      -- Keymaps (set to false to use custom)
      keymaps = {
        chat = '<leader>cc',
        explain = '<leader>ce',
        fix = '<leader>cf',
        generate = '<leader>cg',
        test = '<leader>ct',
        doc = '<leader>cd',
      },
    })
  end,
}
```

---

## Summary

This tutorial has covered everything you need to integrate Claude Code with your development environment:

1. **Introduction** - Understanding CLI vs IDE integration and benefits
2. **VS Code Integration** - Complete setup, configuration, and features
3. **JetBrains Integration** - Plugin installation and IDE-specific features
4. **Terminal Editors** - Vim, Neovim, and Emacs integration
5. **User Journey** - Step-by-step guide from installation to customization
6. **Real-World Examples** - Practical workflows for different scenarios
7. **Keyboard Shortcuts** - Complete reference for all platforms
8. **Features Comparison** - When to use CLI, VS Code, or JetBrains
9. **Advanced Integration** - Custom commands and workspace settings
10. **Troubleshooting** - Solutions for common issues
11. **Pros and Cons** - Making the right choice for your workflow
12. **When to Use Which** - Decision guide and hybrid approaches
13. **Team Setup** - Standardizing configuration across teams
14. **Quick Reference** - Cheat sheets and templates

**Key Takeaways:**

- **CLI** is best for automation, scripting, and remote development
- **VS Code** offers the best balance of features and resource usage for most developers
- **JetBrains** provides the deepest integration for enterprise Java/Kotlin development
- **Terminal editors** work well with CLI in split-pane configurations
- **Hybrid approaches** combine the strengths of multiple tools

Start with your primary IDE's integration, learn the basic shortcuts, and gradually explore advanced features as your workflow demands.

---

*Tutorial 10 of the Claude Code Tutorial Series*
