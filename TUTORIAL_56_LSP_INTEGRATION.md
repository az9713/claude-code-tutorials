# Tutorial 56: LSP Integration & Language Servers

> **IDE-like code navigation in the terminal with Language Server Protocol**

## Table of Contents

1. [Overview](#overview)
2. [What is LSP?](#what-is-lsp)
3. [Supported Languages](#supported-languages)
4. [Getting Started](#getting-started)
5. [Go to Definition](#go-to-definition)
6. [Find References](#find-references)
7. [Hover Documentation](#hover-documentation)
8. [Symbol Search](#symbol-search)
9. [Configuration](#configuration)
10. [Troubleshooting](#troubleshooting)
11. [Best Practices](#best-practices)
12. [Quick Reference](#quick-reference)

---

## Overview

Starting with Claude Code v2.0.74, the LSP (Language Server Protocol) tool enables IDE-like code navigation directly in the terminal. This allows Claude to:

- **Go to definition** - Jump to where symbols are defined
- **Find references** - Locate all usages of a symbol
- **Hover documentation** - Get type info and documentation
- **Symbol search** - Find symbols across the codebase

### Why LSP Matters

Before LSP, Claude relied on regex-based search (Grep/Glob) to understand code structure. While powerful, this approach has limitations:

| Method | Accuracy | Context | Speed |
|--------|----------|---------|-------|
| Grep | Pattern-based | None | Fast |
| LSP | Semantic | Full | Moderate |

LSP provides **semantic understanding** - it knows the difference between a function call, a definition, and a comment with the same text.

---

## What is LSP?

The Language Server Protocol is a standardized interface between development tools and language servers. Language servers provide:

- Code completion
- Go to definition
- Find references
- Hover information
- Diagnostics (errors, warnings)
- Symbol search

### How Claude Code Uses LSP

```
┌─────────────────┐      LSP Protocol      ┌─────────────────┐
│  Claude Code    │◄─────────────────────► │ Language Server │
│  (LSP Client)   │                         │  (e.g., tsserver)│
└─────────────────┘                         └─────────────────┘
        │                                           │
        │ Requests:                                 │ Analysis:
        │ - definition                              │ - Parse AST
        │ - references                              │ - Type checking
        │ - hover                                   │ - Symbol tables
        │ - symbols                                 │ - Dependency graph
        └───────────────────────────────────────────┘
```

Claude Code acts as an LSP client, communicating with language servers installed on your system.

---

## Supported Languages

LSP support depends on having the appropriate language server installed:

### Tier 1: Full Support (Recommended)

| Language | Server | Installation |
|----------|--------|--------------|
| TypeScript/JavaScript | `typescript-language-server` | `npm i -g typescript-language-server` |
| Python | `pylsp` or `pyright` | `pip install python-lsp-server` or `npm i -g pyright` |
| Rust | `rust-analyzer` | Via `rustup component add rust-analyzer` |
| Go | `gopls` | `go install golang.org/x/tools/gopls@latest` |

### Tier 2: Good Support

| Language | Server | Installation |
|----------|--------|--------------|
| Java | `jdtls` | Eclipse JDT Language Server |
| C/C++ | `clangd` | Via LLVM/Clang |
| C# | `omnisharp` | OmniSharp |
| PHP | `intelephense` | `npm i -g intelephense` |
| Ruby | `solargraph` | `gem install solargraph` |

### Tier 3: Basic Support

| Language | Server |
|----------|--------|
| HTML/CSS | `vscode-html-languageserver` |
| JSON | `vscode-json-languageserver` |
| YAML | `yaml-language-server` |
| Bash | `bash-language-server` |

---

## Getting Started

### Step 1: Verify Language Server Installation

```bash
# Check if TypeScript server is available
which typescript-language-server

# Check Python LSP
which pylsp
# or
which pyright

# Check Rust Analyzer
which rust-analyzer
```

### Step 2: Start Claude Code in Your Project

```bash
cd /path/to/your/project
claude
```

### Step 3: Use LSP Commands

```
You: Go to the definition of getUserById

Claude: [Using LSP tool]

Definition found:
📍 src/services/userService.ts:45

export async function getUserById(id: string): Promise<User> {
  return await db.users.findById(id);
}
```

---

## Go to Definition

### Basic Usage

```
You: Go to the definition of handleSubmit

Claude: [Using LSP go-to-definition for "handleSubmit"]

Definition found at src/components/Form.tsx:23

const handleSubmit = async (data: FormData) => {
  await api.submit(data);
  onSuccess();
};
```

### From a Specific Location

```
You: What's the definition of User at line 15 in api.ts?

Claude: [Using LSP go-to-definition at api.ts:15]

"User" is defined in src/types/models.ts:8

interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}
```

### Following Import Chains

```
You: Where does the Router import come from?

Claude: [Using LSP go-to-definition for Router import]

Import chain:
1. src/app.tsx imports from 'react-router-dom'
2. Resolves to: node_modules/react-router-dom/dist/index.d.ts:42

export declare const Router: React.FC<RouterProps>;
```

### Definition for External Libraries

```
You: Show me the definition of useState

Claude: [Using LSP go-to-definition for useState]

Definition in @types/react (node_modules/@types/react/index.d.ts:1234):

function useState<S>(
  initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>];
```

---

## Find References

### Basic Usage

```
You: Find all references to fetchUsers

Claude: [Using LSP find-references for "fetchUsers"]

Found 7 references:

1. src/services/api.ts:23 (definition)
   export async function fetchUsers() { ... }

2. src/components/UserList.tsx:45
   const users = await fetchUsers();

3. src/components/Dashboard.tsx:78
   useEffect(() => { fetchUsers(); }, []);

4. src/hooks/useUsers.ts:12
   return useQuery('users', fetchUsers);

5. tests/api.test.ts:34
   expect(fetchUsers).toHaveBeenCalled();

6. tests/api.test.ts:56
   const result = await fetchUsers();

7. docs/api.md:12 (comment)
   // Use fetchUsers() to get all users
```

### Filtering References

```
You: Find usages of validateEmail in test files only

Claude: [Using LSP find-references + filtering]

References in test files:

1. tests/validation.test.ts:23
   expect(validateEmail('test@example.com')).toBe(true);

2. tests/validation.test.ts:28
   expect(validateEmail('invalid')).toBe(false);

3. tests/form.test.ts:67
   jest.spyOn(validators, 'validateEmail');
```

### Renaming Preparation

```
You: I want to rename handleClick to onButtonClick.
     Show me all places that would need to change.

Claude: [Using LSP find-references for "handleClick"]

Found 4 references that would need updating:

1. src/components/Button.tsx:12 (definition)
   const handleClick = () => { ... }

2. src/components/Button.tsx:34 (JSX usage)
   onClick={handleClick}

3. src/components/Button.tsx:45 (callback reference)
   useCallback(handleClick, [])

4. tests/Button.test.tsx:23
   fireEvent.click(button); // calls handleClick

Shall I perform this rename?
```

---

## Hover Documentation

### Type Information

```
You: What's the type of the response variable in api.ts:45?

Claude: [Using LSP hover at api.ts:45]

Type information:
const response: AxiosResponse<User[]>

Where AxiosResponse is:
{
  data: User[];
  status: number;
  statusText: string;
  headers: AxiosHeaders;
  config: InternalAxiosRequestConfig;
}
```

### Function Signatures

```
You: What parameters does createUser accept?

Claude: [Using LSP hover for createUser]

Function signature:
async function createUser(
  data: CreateUserInput,
  options?: CreateOptions
): Promise<User>

Parameters:
- data: CreateUserInput (required)
  {
    name: string;
    email: string;
    password: string;
  }
- options: CreateOptions (optional)
  {
    sendEmail?: boolean;
    role?: 'user' | 'admin';
  }

Returns: Promise<User>
```

### Documentation Comments

```
You: What's the documentation for formatCurrency?

Claude: [Using LSP hover for formatCurrency]

/**
 * Formats a number as currency string.
 *
 * @param amount - The numeric amount to format
 * @param currency - ISO 4217 currency code (default: 'USD')
 * @param locale - BCP 47 language tag (default: 'en-US')
 * @returns Formatted currency string
 *
 * @example
 * formatCurrency(1234.56) // "$1,234.56"
 * formatCurrency(1234.56, 'EUR', 'de-DE') // "1.234,56 €"
 */
function formatCurrency(
  amount: number,
  currency?: string,
  locale?: string
): string
```

---

## Symbol Search

### Finding Symbols by Name

```
You: Find all exports named "config" in the codebase

Claude: [Using LSP workspace-symbols for "config"]

Found symbols:

1. src/config/index.ts
   export const config: AppConfig

2. src/config/database.ts
   export const config: DatabaseConfig

3. src/testing/config.ts
   export const config: TestConfig

4. packages/shared/config.ts
   export interface ConfigOptions
```

### Symbol Types

LSP can distinguish between symbol types:

| Type | Examples |
|------|----------|
| Function | `function fetchUser()` |
| Class | `class UserService` |
| Interface | `interface User` |
| Variable | `const API_URL` |
| Type | `type UserId = string` |
| Enum | `enum Status` |
| Module | `namespace Utils` |

### Searching by Type

```
You: Find all interfaces in the types directory

Claude: [Using LSP workspace-symbols with kind filter]

Interfaces in src/types/:

1. src/types/user.ts:5
   interface User

2. src/types/user.ts:15
   interface CreateUserInput

3. src/types/api.ts:3
   interface ApiResponse<T>

4. src/types/api.ts:12
   interface PaginatedResponse<T>

5. src/types/config.ts:1
   interface AppConfig
```

---

## Configuration

### Settings

Configure LSP behavior in `.claude/settings.json`:

```json
{
  "lsp": {
    "enabled": true,
    "servers": {
      "typescript": {
        "command": "typescript-language-server",
        "args": ["--stdio"],
        "filetypes": ["typescript", "typescriptreact", "javascript", "javascriptreact"]
      },
      "python": {
        "command": "pylsp",
        "args": [],
        "filetypes": ["python"]
      },
      "rust": {
        "command": "rust-analyzer",
        "args": [],
        "filetypes": ["rust"]
      }
    },
    "timeout": 5000,
    "maxFileSize": 1000000
  }
}
```

### Language-Specific Settings

```json
{
  "lsp": {
    "servers": {
      "typescript": {
        "command": "typescript-language-server",
        "args": ["--stdio"],
        "initializationOptions": {
          "preferences": {
            "importModuleSpecifierPreference": "relative"
          }
        }
      }
    }
  }
}
```

### Disabling LSP

```json
{
  "lsp": {
    "enabled": false
  }
}
```

Or per-project in `.claude/settings.json`:

```json
{
  "lsp": {
    "enabled": false
  }
}
```

---

## Troubleshooting

### Language Server Not Found

**Symptom**: "No language server available for this file type"

**Solutions**:

1. Install the appropriate language server
   ```bash
   # TypeScript
   npm install -g typescript-language-server typescript

   # Python
   pip install python-lsp-server

   # Rust
   rustup component add rust-analyzer
   ```

2. Verify the server is in PATH
   ```bash
   which typescript-language-server
   ```

3. Configure custom server path in settings
   ```json
   {
     "lsp": {
       "servers": {
         "typescript": {
           "command": "/custom/path/typescript-language-server"
         }
       }
     }
   }
   ```

### Slow Response Times

**Symptom**: LSP operations take too long

**Solutions**:

1. Increase timeout in settings
   ```json
   {
     "lsp": {
       "timeout": 10000
     }
   }
   ```

2. Exclude large directories
   ```json
   {
     "lsp": {
       "excludePatterns": ["**/node_modules/**", "**/dist/**"]
     }
   }
   ```

3. Ensure project has proper config
   - `tsconfig.json` for TypeScript
   - `pyrightconfig.json` or `setup.py` for Python
   - `Cargo.toml` for Rust

### Incorrect Results

**Symptom**: Wrong definition or missing references

**Solutions**:

1. Ensure project is properly configured
   ```bash
   # TypeScript: check tsconfig.json exists and is valid
   npx tsc --noEmit

   # Python: ensure virtual environment is activated
   source venv/bin/activate
   ```

2. Check for compilation errors
   - LSP may give incorrect results if project has errors

3. Restart the language server
   - Close and reopen Claude Code session

### No Results Found

**Symptom**: "No definition found" or "No references found"

**Solutions**:

1. Check file is saved (LSP uses disk state)

2. Verify symbol name is correct (case-sensitive)

3. Check if symbol is in scope
   - Private/unexported symbols may not be found across files

4. For dynamic languages, LSP may not find all usages
   - Fall back to Grep for comprehensive search

---

## Best Practices

### 1. Use LSP for Precise Navigation

```
# Good: Semantic understanding
You: Go to the definition of createUser

# Works but less precise
You: Search for "function createUser"
```

### 2. Combine LSP with Grep for Completeness

```
You: Find all references to handleSubmit, including comments

Claude: [Uses LSP for code references]
[Uses Grep for comments and strings]

Code references (LSP): 5 found
Additional mentions (Grep): 2 in comments
```

### 3. Use Hover for Quick Type Checking

```
You: What's the return type of this function before I modify it?

Claude: [Uses LSP hover]

Current return type: Promise<User | null>
```

### 4. Verify Refactoring Scope

```
You: Before renaming, show me everywhere this will change

Claude: [Uses LSP find-references]

This rename will affect 12 files, 34 locations.
```

### 5. Check Documentation Before Implementation

```
You: What does this library function expect?

Claude: [Uses LSP hover to get JSDoc/docstring]

According to the documentation:
- Required: id (string)
- Optional: options (object with timeout, retries)
- Throws: NotFoundError if id doesn't exist
```

---

## Quick Reference

### LSP Operations

| Operation | Use Case | Example |
|-----------|----------|---------|
| Go to Definition | Find where symbol is declared | "Where is User defined?" |
| Find References | Find all usages of symbol | "Where is fetchUser called?" |
| Hover | Get type info and docs | "What type is this variable?" |
| Symbol Search | Find symbols by name | "Find all exports named config" |

### Common Commands

```
# Go to definition
"Go to the definition of [symbol]"
"Where is [symbol] defined?"
"Show me the implementation of [function]"

# Find references
"Find all references to [symbol]"
"Where is [function] called?"
"Show usages of [variable]"

# Hover
"What's the type of [variable]?"
"Show documentation for [function]"
"What parameters does [function] take?"

# Symbol search
"Find all classes named Service"
"List all interfaces in src/types"
"Find exports named config"
```

### When to Use LSP vs Grep

| Task | Use LSP | Use Grep |
|------|---------|----------|
| Find definition | ✓ | |
| Find all usages (code) | ✓ | |
| Find mentions in comments | | ✓ |
| Find in strings/templates | | ✓ |
| Complex regex patterns | | ✓ |
| Cross-language search | | ✓ |
| Dynamic code references | | ✓ |

---

## Sources

- [Language Server Protocol Specification](https://microsoft.github.io/language-server-protocol/)
- [Claude Code Changelog v2.0.74](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
