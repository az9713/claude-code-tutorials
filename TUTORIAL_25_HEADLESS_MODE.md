# Tutorial 25: Headless Mode in Claude Code

## Complete Guide to Non-Interactive Scripting and Automation

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Understanding Headless Mode](#understanding-headless-mode)
5. [Basic Headless Usage](#basic-headless-usage)
6. [Scripting with Claude Code](#scripting-with-claude-code)
7. [Advanced Headless Patterns](#advanced-headless-patterns)
8. [Real-World Examples](#real-world-examples)
9. [Output Format Comparison](#output-format-comparison)
10. [Exit Codes Reference](#exit-codes-reference)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Overview

### What is Headless Mode?

Headless mode is a way to run Claude Code without an interactive terminal session.
Instead of having a back-and-forth conversation, you send a single prompt and receive
a single response. This makes Claude Code behave like a traditional command-line tool
that can be integrated into scripts, pipelines, and automated workflows.

Think of it like the difference between:
- **Interactive mode**: Having a phone call with someone (back-and-forth conversation)
- **Headless mode**: Sending a text message and getting a reply (single exchange)

### Why Use Headless Mode?

Headless mode unlocks powerful automation capabilities:

1. **Scripting**: Write bash, Python, or other scripts that leverage Claude's AI
2. **CI/CD Integration**: Add AI-powered code review to your build pipelines
3. **Batch Processing**: Process hundreds of files automatically
4. **Data Pipelines**: Chain Claude with other command-line tools
5. **Scheduled Tasks**: Run AI analysis on a cron schedule
6. **Testing**: Automate test generation and validation

### Key Characteristics

| Aspect | Interactive Mode | Headless Mode |
|--------|------------------|---------------|
| Input | Keyboard typing | Command arguments or stdin |
| Output | Terminal display | stdout (capturable) |
| Conversation | Multi-turn | Single prompt/response |
| Use case | Development work | Automation |
| Human presence | Required | Not required |

---

## Prerequisites

Before diving into headless mode, ensure you have the following set up:

### 1. Claude Code Installation

Claude Code must be installed and accessible from your command line:

```bash
# Verify installation
claude --version

# Expected output: something like "claude-code 1.x.x"
```

If not installed, install via npm:

```bash
npm install -g @anthropic-ai/claude-code
```

### 2. API Key Configuration

Headless mode requires a valid Anthropic API key:

```bash
# Option 1: Environment variable (recommended for scripts)
export ANTHROPIC_API_KEY="YOUR_API_KEY"

# Option 2: Check if already configured
claude config get apiKey
```

### 3. Shell Scripting Knowledge

Basic familiarity with:
- Bash scripting (variables, conditionals, loops)
- Command-line piping (`|`, `>`, `>>`)
- Exit codes and error handling
- Process substitution

### 4. Recommended Tools

For advanced usage, these tools are helpful:
- `jq` - JSON parsing
- `xargs` - Parallel execution
- `tee` - Output duplication
- `timeout` - Command timeouts

---

## Core Concepts

### The Three Pillars of Headless Mode

Understanding headless mode requires grasping three fundamental concepts:

```
+-------------------+     +-------------------+     +-------------------+
|   --print Flag    | --> |  Output Formats   | --> |     Piping        |
+-------------------+     +-------------------+     +-------------------+
|                   |     |                   |     |                   |
| Enables headless  |     | Controls response |     | Integrates with   |
| execution         |     | structure         |     | other tools       |
|                   |     |                   |     |                   |
+-------------------+     +-------------------+     +-------------------+
```

### The --print Flag

The `--print` flag (or `-p`) is what activates headless mode:

```bash
# Interactive mode (no --print)
claude
# Opens interactive session

# Headless mode (with --print)
claude --print "Hello, Claude!"
# Prints response and exits
```

### Output Formats

Control how Claude formats its response:

```bash
# Plain text (default)
claude --print "What is 2+2?"
# Output: 4

# JSON format
claude --print "What is 2+2?" --output-format json
# Output: {"result": "4", "explanation": "..."}

# Markdown format
claude --print "Explain variables" --output-format markdown
# Output: # Variables\n\nVariables are...
```

### Piping

Send data to Claude and chain with other commands:

```bash
# Pipe input
cat file.txt | claude --print "Summarize this"

# Pipe output
claude --print "List 5 items" | grep "item"

# Full pipeline
cat code.py | claude --print "Review this" | tee review.md
```

---

## Understanding Headless Mode

### What Headless Mode Enables

Headless mode transforms Claude Code from an interactive assistant into a powerful
automation component. Here's what becomes possible:

#### 1. Script Integration

```bash
#!/bin/bash
# Your script can now "ask Claude" for help

USER_INPUT="What's the weather like?"
RESPONSE=$(claude --print "As a helpful assistant, respond to: $USER_INPUT")
echo "Claude says: $RESPONSE"
```

#### 2. Automated Decision Making

```bash
#!/bin/bash
# Let Claude analyze and decide

CODE_QUALITY=$(claude --print "Rate this code 1-10, respond with just the number:
$(cat mycode.py)")

if [ "$CODE_QUALITY" -lt 5 ]; then
    echo "Code needs improvement"
    exit 1
fi
```

#### 3. Data Transformation

```bash
# Convert between formats
cat data.csv | claude --print "Convert this CSV to JSON"

# Extract information
cat report.pdf | claude --print "Extract all dates mentioned"
```

### Use Cases for Headless Mode

#### Scripts and Automation

```bash
#!/bin/bash
# daily_report.sh - Generate daily project summary

DATE=$(date +%Y-%m-%d)
GIT_LOG=$(git log --since="yesterday" --oneline)

SUMMARY=$(claude --print "Summarize these git commits for a daily standup:
$GIT_LOG")

echo "# Daily Report - $DATE" > "reports/$DATE.md"
echo "$SUMMARY" >> "reports/$DATE.md"
```

#### CI/CD Pipelines

```yaml
# GitLab CI example
code_review:
  stage: test
  script:
    - |
      DIFF=$(git diff origin/main)
      claude --print "Review this diff for issues: $DIFF" > review.txt
    - cat review.txt
  artifacts:
    paths:
      - review.txt
```

#### Scheduled Tasks

```bash
# Crontab entry: Run every Monday at 9 AM
0 9 * * 1 /path/to/weekly_code_analysis.sh

# weekly_code_analysis.sh
#!/bin/bash
cd /project
ANALYSIS=$(claude --print "Analyze this codebase for technical debt:
$(find . -name '*.js' -exec cat {} \;)")
mail -s "Weekly Code Analysis" team@company.com <<< "$ANALYSIS"
```

### Non-Interactive Execution

Understanding non-interactive execution is crucial:

```
INTERACTIVE SESSION:
+--------+                    +--------+
|  User  | <-- Type prompt -> | Claude |
+--------+                    +--------+
          <-- See response --
          <-- Type follow-up ->
          <-- See response --
          ...continues...

NON-INTERACTIVE (HEADLESS):
+--------+                    +--------+
| Script | -- Send prompt --> | Claude |
+--------+                    +--------+
          <-- Get response --
          (Exit immediately)
```

Key differences:
- No conversation history maintained between calls
- Each invocation is independent
- Output goes to stdout (not formatted terminal)
- Errors go to stderr
- Returns exit code for script control flow

---

## Basic Headless Usage

### The --print Flag

The `--print` flag is your gateway to headless mode. Let's explore it thoroughly.

#### Basic Syntax

```bash
claude --print "Your prompt here"

# Or use the short form
claude -p "Your prompt here"
```

#### Simple Examples

```bash
# Ask a question
claude --print "What is the capital of France?"
# Output: The capital of France is Paris.

# Request information
claude --print "List the planets in our solar system"
# Output: Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune

# Get code
claude --print "Write a Python hello world program"
# Output: print("Hello, World!")
```

#### Prompt Quoting

Be careful with special characters:

```bash
# Single quotes - literals preserved
claude --print 'What does $PATH mean in bash?'

# Double quotes - variables expanded
NAME="Alice"
claude --print "Say hello to $NAME"

# Escaping special characters
claude --print "What does \`backtick\` do in bash?"

# Heredoc for complex prompts
claude --print "$(cat <<'EOF'
Analyze this code:
```python
def hello():
    print("Hello, World!")
```
List any issues found.
EOF
)"
```

#### Multiline Prompts

```bash
# Using heredoc (recommended for complex prompts)
claude --print "$(cat <<'EOF'
You are a code reviewer.

Review the following function for:
1. Correctness
2. Performance
3. Style

```javascript
function sum(arr) {
    let total = 0;
    for (let i = 0; i <= arr.length; i++) {
        total += arr[i];
    }
    return total;
}
```
EOF
)"

# Using echo with newlines
claude --print "$(echo -e "Line 1\nLine 2\nLine 3")"
```

### Output Formats

Claude Code supports multiple output formats for different use cases.

#### Default Text Format

```bash
claude --print "Explain what an API is"

# Output (plain text):
# An API (Application Programming Interface) is a set of protocols,
# routines, and tools that allow different software applications to
# communicate with each other...
```

Best for:
- Human-readable output
- Simple scripts
- Direct display

#### JSON Format

```bash
claude --print "List 3 programming languages with their main use case" \
    --output-format json

# Output:
# {
#   "languages": [
#     {"name": "Python", "useCase": "Data science and scripting"},
#     {"name": "JavaScript", "useCase": "Web development"},
#     {"name": "Rust", "useCase": "Systems programming"}
#   ]
# }
```

Best for:
- Machine parsing
- Data extraction
- Integration with other tools
- Structured responses

#### Markdown Format

```bash
claude --print "Explain the MVC pattern" --output-format markdown

# Output:
# # MVC Pattern
#
# The **Model-View-Controller** pattern separates an application into three components:
#
# ## Model
# - Handles data and business logic
# - Independent of the user interface
#
# ## View
# - Displays data to the user
# - Receives user input
#
# ## Controller
# - Acts as intermediary
# - Updates Model and View
```

Best for:
- Documentation generation
- Reports
- README files
- Formatted output

### Parsing Responses

#### Parsing JSON with jq

```bash
# Get structured response
RESPONSE=$(claude --print "Return a JSON object with name and age for a fictional person" \
    --output-format json)

# Extract specific fields
NAME=$(echo "$RESPONSE" | jq -r '.name')
AGE=$(echo "$RESPONSE" | jq -r '.age')

echo "Person: $NAME, Age: $AGE"

# Handle arrays
ITEMS=$(claude --print "List 5 fruits as a JSON array" --output-format json)
echo "$ITEMS" | jq -r '.[]'
# apple
# banana
# orange
# grape
# mango
```

#### Parsing Text with grep/sed/awk

```bash
# Extract specific lines
claude --print "List 5 items, one per line" | grep "item"

# Get first line only
claude --print "Give a one-line summary then details" | head -1

# Extract numbered items
claude --print "List 3 benefits" | grep -E "^[0-9]"

# Transform output
claude --print "List colors" | tr ',' '\n'
```

---

## Scripting with Claude Code

### Shell Script Integration

#### Capturing Output

```bash
#!/bin/bash
# Capturing Claude's response into a variable

# Basic capture
RESPONSE=$(claude --print "What is 2 + 2?")
echo "Claude said: $RESPONSE"

# Capture with error handling
if RESPONSE=$(claude --print "Explain recursion" 2>&1); then
    echo "Success: $RESPONSE"
else
    echo "Error occurred: $RESPONSE"
    exit 1
fi

# Capture to file
claude --print "Write a poem about coding" > poem.txt

# Capture and display simultaneously
claude --print "Generate a report" | tee report.txt
```

#### Error Handling

```bash
#!/bin/bash
# Robust error handling for Claude Code scripts

set -e  # Exit on any error

# Function to call Claude with retry logic
ask_claude() {
    local prompt="$1"
    local max_retries=3
    local retry_count=0

    while [ $retry_count -lt $max_retries ]; do
        if response=$(claude --print "$prompt" 2>&1); then
            echo "$response"
            return 0
        else
            retry_count=$((retry_count + 1))
            echo "Attempt $retry_count failed, retrying..." >&2
            sleep 2
        fi
    done

    echo "Failed after $max_retries attempts" >&2
    return 1
}

# Usage
RESULT=$(ask_claude "What is the meaning of life?")
echo "$RESULT"
```

#### Comprehensive Error Handler

```bash
#!/bin/bash
# claude_wrapper.sh - Production-ready Claude wrapper

claude_safe() {
    local prompt="$1"
    local output_format="${2:-text}"
    local timeout_seconds="${3:-60}"

    # Validate inputs
    if [ -z "$prompt" ]; then
        echo "Error: Prompt required" >&2
        return 2
    fi

    # Check API key
    if [ -z "$ANTHROPIC_API_KEY" ]; then
        echo "Error: ANTHROPIC_API_KEY not set" >&2
        return 3
    fi

    # Execute with timeout
    local result
    local exit_code

    result=$(timeout "$timeout_seconds" claude --print "$prompt" \
        --output-format "$output_format" 2>&1)
    exit_code=$?

    case $exit_code in
        0)
            echo "$result"
            return 0
            ;;
        124)
            echo "Error: Request timed out after ${timeout_seconds}s" >&2
            return 4
            ;;
        *)
            echo "Error: Claude returned exit code $exit_code" >&2
            echo "Details: $result" >&2
            return $exit_code
            ;;
    esac
}

# Export for use in other scripts
export -f claude_safe
```

#### Exit Codes

```bash
#!/bin/bash
# Understanding and using exit codes

claude --print "Hello"
EXIT_CODE=$?

case $EXIT_CODE in
    0)
        echo "Success!"
        ;;
    1)
        echo "General error occurred"
        ;;
    2)
        echo "Invalid arguments provided"
        ;;
    3)
        echo "API error (authentication, rate limit, etc.)"
        ;;
    *)
        echo "Unknown error: $EXIT_CODE"
        ;;
esac

# Use in conditionals
if claude --print "Test prompt" > /dev/null 2>&1; then
    echo "Claude is working"
else
    echo "Claude is not available"
fi
```

### Pipeline Integration

#### Piping Input to Claude

```bash
# Pipe file contents
cat README.md | claude --print "Summarize this document"

# Pipe command output
ls -la | claude --print "Explain what these files might be"

# Pipe multiple files
cat src/*.py | claude --print "Find any security vulnerabilities"

# Pipe with preprocessing
cat data.json | jq '.items[]' | claude --print "Analyze these items"
```

#### Piping Output from Claude

```bash
# Filter Claude's output
claude --print "List 10 random words" | grep -E "^[a-z]+"

# Count lines
claude --print "List programming languages" | wc -l

# Sort output
claude --print "List fruits randomly" | sort

# Unique values
claude --print "Generate 20 random colors" | sort | uniq
```

#### Building Data Pipelines

```bash
#!/bin/bash
# Complex pipeline example

# Pipeline 1: Code analysis pipeline
find . -name "*.py" -type f \
    | xargs cat \
    | claude --print "Identify the main classes and their responsibilities" \
    | tee analysis.md \
    | claude --print "Create a summary table from this analysis" \
    > summary.md

# Pipeline 2: Log analysis
cat /var/log/app.log \
    | tail -1000 \
    | claude --print "Identify error patterns and their frequency" \
    | claude --print "Suggest fixes for these errors" --output-format json \
    | jq '.suggestions[]' \
    > fixes.json

# Pipeline 3: Documentation pipeline
cat src/main.ts \
    | claude --print "Extract all function signatures" \
    | claude --print "Generate API documentation in markdown" \
    | claude --print "Add usage examples for each function" \
    > API_DOCS.md
```

#### Stdin Processing

```bash
# Process stdin directly
echo "Hello, World!" | claude --print "Translate to Spanish"

# Here-string
claude --print "Analyze this" <<< "Some text to analyze"

# Here-document
claude --print "Review this configuration" << 'EOF'
{
    "database": {
        "host": "localhost",
        "port": 5432
    }
}
EOF
```

---

## Advanced Headless Patterns

### Multi-Turn Scripted Conversations

While headless mode is typically single-turn, you can simulate conversations.

#### Using Sessions in Scripts

```bash
#!/bin/bash
# Simulating a conversation with context

# Store conversation context
CONTEXT=""

ask_with_context() {
    local question="$1"

    # Build prompt with context
    local full_prompt="Previous conversation:
$CONTEXT

Current question: $question

Respond to the current question while considering the context."

    # Get response
    local response=$(claude --print "$full_prompt")

    # Update context
    CONTEXT="$CONTEXT
User: $question
Assistant: $response"

    echo "$response"
}

# Usage
ask_with_context "My name is Alice"
ask_with_context "What's my name?"  # Should remember "Alice"
ask_with_context "Tell me a joke about my name"
```

#### Session Files

```bash
#!/bin/bash
# Using files to maintain conversation state

SESSION_FILE="/tmp/claude_session_$$"

start_session() {
    echo "" > "$SESSION_FILE"
}

add_to_session() {
    local role="$1"
    local content="$2"
    echo "[$role]: $content" >> "$SESSION_FILE"
}

ask_in_session() {
    local question="$1"

    # Read existing session
    local context=$(cat "$SESSION_FILE")

    # Build prompt
    local prompt="You are having a conversation. Here is the history:
$context

User: $question

Respond naturally, remembering previous context."

    # Get response
    local response=$(claude --print "$prompt")

    # Save to session
    add_to_session "User" "$question"
    add_to_session "Assistant" "$response"

    echo "$response"
}

end_session() {
    rm -f "$SESSION_FILE"
}

# Usage
start_session
ask_in_session "Let's talk about Python"
ask_in_session "What are its main features?"
ask_in_session "Compare it to JavaScript"
end_session
```

#### Automated Workflows

```bash
#!/bin/bash
# Automated code improvement workflow

improve_code() {
    local file="$1"
    local code=$(cat "$file")

    echo "Step 1: Analyzing code..."
    ANALYSIS=$(claude --print "Analyze this code for issues:
\`\`\`
$code
\`\`\`
Return a numbered list of issues.")

    echo "Step 2: Generating improvements..."
    IMPROVED=$(claude --print "Here's code with these issues:

Issues found:
$ANALYSIS

Original code:
\`\`\`
$code
\`\`\`

Fix all the issues and return only the improved code.")

    echo "Step 3: Validating improvements..."
    VALIDATION=$(claude --print "Verify this improved code is correct:
\`\`\`
$IMPROVED
\`\`\`
Return 'VALID' if correct, or list remaining issues.")

    if [[ "$VALIDATION" == *"VALID"* ]]; then
        echo "$IMPROVED" > "$file"
        echo "Code improved successfully!"
    else
        echo "Validation failed: $VALIDATION"
    fi
}

improve_code "src/main.py"
```

### Batch Processing

#### Processing Multiple Files

```bash
#!/bin/bash
# batch_process.sh - Process multiple files with Claude

process_file() {
    local file="$1"
    local operation="$2"

    echo "Processing: $file"

    local content=$(cat "$file")
    local result=$(claude --print "$operation

File: $file
Content:
\`\`\`
$content
\`\`\`")

    echo "$result"
}

# Process all Python files
for file in src/*.py; do
    process_file "$file" "Add docstrings to all functions"
done

# Process specific file types
find . -name "*.js" -type f | while read file; do
    process_file "$file" "Convert to TypeScript" > "${file%.js}.ts"
done
```

#### Parallel Execution

```bash
#!/bin/bash
# parallel_process.sh - Process files in parallel

# Using GNU parallel
find src -name "*.py" | parallel -j 4 '
    echo "Processing {}"
    claude --print "Review this file: $(cat {})" > "{}.review"
'

# Using xargs
find src -name "*.py" | xargs -P 4 -I {} bash -c '
    claude --print "Analyze: $(cat {})" > "{}.analysis"
'

# Using background jobs with job control
MAX_JOBS=4
job_count=0

for file in src/*.py; do
    (
        claude --print "Process: $(cat "$file")" > "$file.processed"
    ) &

    job_count=$((job_count + 1))
    if [ $job_count -ge $MAX_JOBS ]; then
        wait -n
        job_count=$((job_count - 1))
    fi
done
wait  # Wait for remaining jobs
```

#### Progress Tracking

```bash
#!/bin/bash
# progress_batch.sh - Batch processing with progress

FILES=(src/*.py)
TOTAL=${#FILES[@]}
CURRENT=0
ERRORS=0

progress_bar() {
    local current=$1
    local total=$2
    local width=50
    local percentage=$((current * 100 / total))
    local filled=$((current * width / total))
    local empty=$((width - filled))

    printf "\r["
    printf "%${filled}s" | tr ' ' '#'
    printf "%${empty}s" | tr ' ' '-'
    printf "] %d%% (%d/%d)" $percentage $current $total
}

log_file="batch_$(date +%Y%m%d_%H%M%S).log"

for file in "${FILES[@]}"; do
    CURRENT=$((CURRENT + 1))
    progress_bar $CURRENT $TOTAL

    if result=$(claude --print "Process: $(cat "$file")" 2>&1); then
        echo "$result" > "$file.output"
        echo "[SUCCESS] $file" >> "$log_file"
    else
        echo "[ERROR] $file: $result" >> "$log_file"
        ERRORS=$((ERRORS + 1))
    fi
done

echo ""
echo "Complete! $ERRORS errors. See $log_file for details."
```

---

## Real-World Examples

### Example 1: Automated Code Review in CI/CD

```bash
#!/bin/bash
# ci_code_review.sh - Automated code review for CI/CD pipelines

set -e

# Configuration
BASE_BRANCH="${BASE_BRANCH:-main}"
OUTPUT_FILE="code_review.md"

echo "# Automated Code Review" > "$OUTPUT_FILE"
echo "Generated: $(date)" >> "$OUTPUT_FILE"
echo "" >> "$OUTPUT_FILE"

# Get changed files
CHANGED_FILES=$(git diff --name-only "$BASE_BRANCH"...HEAD | grep -E '\.(js|ts|py|go)$' || true)

if [ -z "$CHANGED_FILES" ]; then
    echo "No code files changed." >> "$OUTPUT_FILE"
    exit 0
fi

# Review each file
for file in $CHANGED_FILES; do
    if [ ! -f "$file" ]; then
        continue
    fi

    echo "Reviewing: $file"
    echo "## $file" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"

    # Get the diff for this file
    DIFF=$(git diff "$BASE_BRANCH"...HEAD -- "$file")

    # Request review
    REVIEW=$(claude --print "You are a senior code reviewer. Review this diff:

\`\`\`diff
$DIFF
\`\`\`

Provide:
1. **Summary**: What does this change do?
2. **Issues**: Any bugs, security issues, or problems?
3. **Suggestions**: How could this be improved?
4. **Rating**: 1-5 stars

Be concise but thorough." --output-format markdown)

    echo "$REVIEW" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "---" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
done

# Generate summary
echo "## Overall Summary" >> "$OUTPUT_FILE"

SUMMARY=$(claude --print "Summarize this code review in 2-3 sentences:

$(cat "$OUTPUT_FILE")")

echo "$SUMMARY" >> "$OUTPUT_FILE"

echo "Review complete: $OUTPUT_FILE"
```

### Example 2: Batch Documentation Generator

```bash
#!/bin/bash
# generate_docs.sh - Generate documentation for all source files

SOURCE_DIR="${1:-.}"
OUTPUT_DIR="${2:-docs}"
FILE_PATTERN="${3:-*.py}"

mkdir -p "$OUTPUT_DIR"

# Generate index
INDEX_FILE="$OUTPUT_DIR/INDEX.md"
echo "# API Documentation" > "$INDEX_FILE"
echo "" >> "$INDEX_FILE"
echo "Generated: $(date)" >> "$INDEX_FILE"
echo "" >> "$INDEX_FILE"
echo "## Modules" >> "$INDEX_FILE"
echo "" >> "$INDEX_FILE"

# Process each file
find "$SOURCE_DIR" -name "$FILE_PATTERN" -type f | while read file; do
    # Skip test files
    if [[ "$file" == *"test"* ]]; then
        continue
    fi

    filename=$(basename "$file")
    module_name="${filename%.*}"
    doc_file="$OUTPUT_DIR/${module_name}.md"

    echo "Documenting: $file -> $doc_file"

    # Generate documentation
    DOC=$(claude --print "Generate comprehensive documentation for this Python module:

\`\`\`python
$(cat "$file")
\`\`\`

Include:
1. Module overview
2. Class documentation with attributes and methods
3. Function documentation with parameters and return values
4. Usage examples
5. Notes and warnings

Format as Markdown." --output-format markdown)

    echo "$DOC" > "$doc_file"

    # Add to index
    echo "- [$module_name](./${module_name}.md)" >> "$INDEX_FILE"
done

echo ""
echo "Documentation generated in $OUTPUT_DIR/"
echo "Index: $INDEX_FILE"
```

### Example 3: Test Generation Script

```bash
#!/bin/bash
# generate_tests.sh - Generate unit tests for source files

generate_tests() {
    local source_file="$1"
    local test_file="$2"

    local code=$(cat "$source_file")

    # Detect language
    local ext="${source_file##*.}"
    local framework=""

    case "$ext" in
        py) framework="pytest" ;;
        js) framework="jest" ;;
        ts) framework="jest with TypeScript" ;;
        go) framework="Go testing package" ;;
        *) framework="appropriate testing framework" ;;
    esac

    echo "Generating tests for $source_file using $framework..."

    TESTS=$(claude --print "Generate comprehensive unit tests for this code using $framework:

\`\`\`
$code
\`\`\`

Requirements:
1. Test all public functions/methods
2. Include edge cases
3. Test error conditions
4. Use descriptive test names
5. Include setup/teardown if needed
6. Aim for high coverage

Return only the test code, ready to run.")

    echo "$TESTS" > "$test_file"
    echo "Tests written to $test_file"
}

# Usage examples
# Generate tests for a single file
generate_tests "src/calculator.py" "tests/test_calculator.py"

# Generate tests for all files
for file in src/*.py; do
    filename=$(basename "$file")
    test_file="tests/test_${filename}"
    generate_tests "$file" "$test_file"
done
```

### Example 4: Commit Message Generator

```bash
#!/bin/bash
# commit_with_ai.sh - Generate commit messages with AI

# Get staged changes
STAGED_DIFF=$(git diff --cached)

if [ -z "$STAGED_DIFF" ]; then
    echo "No staged changes. Stage files with 'git add' first."
    exit 1
fi

# Get list of staged files
STAGED_FILES=$(git diff --cached --name-only)

echo "Staged files:"
echo "$STAGED_FILES"
echo ""

# Generate commit message
COMMIT_MSG=$(claude --print "Generate a conventional commit message for these changes:

Files changed:
$STAGED_FILES

Diff:
\`\`\`diff
$STAGED_DIFF
\`\`\`

Requirements:
1. Use conventional commit format: type(scope): description
2. Types: feat, fix, docs, style, refactor, test, chore
3. Keep subject line under 50 characters
4. Add body if changes are complex
5. Reference issues if apparent from code

Return only the commit message, no explanation.")

echo "Suggested commit message:"
echo "------------------------"
echo "$COMMIT_MSG"
echo "------------------------"
echo ""

# Ask for confirmation
read -p "Use this message? (y/n/e for edit): " choice

case "$choice" in
    y|Y)
        git commit -m "$COMMIT_MSG"
        echo "Committed!"
        ;;
    e|E)
        # Open in editor
        echo "$COMMIT_MSG" > /tmp/commit_msg.txt
        ${EDITOR:-vim} /tmp/commit_msg.txt
        git commit -F /tmp/commit_msg.txt
        rm /tmp/commit_msg.txt
        echo "Committed with edited message!"
        ;;
    *)
        echo "Commit cancelled."
        exit 0
        ;;
esac
```

### Example 5: Release Notes Automation

```bash
#!/bin/bash
# generate_release_notes.sh - Generate release notes from git history

VERSION="${1:-$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")}"
PREVIOUS_TAG="${2:-$(git describe --tags --abbrev=0 HEAD~1 2>/dev/null || echo "")}"

OUTPUT_FILE="RELEASE_NOTES_${VERSION}.md"

echo "Generating release notes for $VERSION..."

# Get commits since last tag
if [ -n "$PREVIOUS_TAG" ]; then
    COMMITS=$(git log "$PREVIOUS_TAG"..HEAD --pretty=format:"- %s (%h)" --no-merges)
    FULL_LOG=$(git log "$PREVIOUS_TAG"..HEAD --pretty=format:"%h %s%n%b" --no-merges)
else
    COMMITS=$(git log --pretty=format:"- %s (%h)" --no-merges -50)
    FULL_LOG=$(git log --pretty=format:"%h %s%n%b" --no-merges -50)
fi

# Get contributors
CONTRIBUTORS=$(git log "$PREVIOUS_TAG"..HEAD --pretty=format:"%an" | sort | uniq)

# Generate release notes
NOTES=$(claude --print "Generate professional release notes from these commits:

Version: $VERSION
Previous Version: $PREVIOUS_TAG

Commits:
$COMMITS

Full commit details:
$FULL_LOG

Contributors:
$CONTRIBUTORS

Format the release notes with:
1. **Highlights** - Most important changes (2-3 items)
2. **New Features** - New functionality added
3. **Bug Fixes** - Issues resolved
4. **Improvements** - Enhancements and optimizations
5. **Breaking Changes** - Any breaking changes (if any)
6. **Contributors** - Thank contributors

Use professional tone. Group related changes. Be concise but informative.")

# Write release notes
cat > "$OUTPUT_FILE" << EOF
# Release Notes - $VERSION

Released: $(date +%Y-%m-%d)

$NOTES

---
*Generated with Claude Code*
EOF

echo "Release notes written to $OUTPUT_FILE"

# Optionally create git tag
read -p "Create git tag $VERSION? (y/n): " create_tag
if [ "$create_tag" = "y" ]; then
    git tag -a "$VERSION" -m "Release $VERSION"
    echo "Tag $VERSION created"
fi
```

### Example 6: Code Migration Assistant

```bash
#!/bin/bash
# migrate_code.sh - Assist with code migrations

MIGRATION_TYPE="${1:-}"
SOURCE_DIR="${2:-.}"

case "$MIGRATION_TYPE" in
    "js-to-ts")
        PROMPT="Convert this JavaScript file to TypeScript:
- Add proper type annotations
- Use interfaces for objects
- Handle null/undefined properly
- Keep functionality identical"
        FILE_PATTERN="*.js"
        NEW_EXT=".ts"
        ;;
    "class-to-functional")
        PROMPT="Convert this React class component to a functional component:
- Use hooks (useState, useEffect, etc.)
- Preserve all functionality
- Keep prop types"
        FILE_PATTERN="*.jsx"
        NEW_EXT=".jsx"
        ;;
    "sync-to-async")
        PROMPT="Convert synchronous code to use async/await:
- Add async to functions that need it
- Use await for async operations
- Handle errors with try/catch
- Preserve functionality"
        FILE_PATTERN="*.js"
        NEW_EXT=".js"
        ;;
    *)
        echo "Usage: migrate_code.sh <migration-type> [source-dir]"
        echo ""
        echo "Migration types:"
        echo "  js-to-ts          - JavaScript to TypeScript"
        echo "  class-to-functional - React class to functional components"
        echo "  sync-to-async     - Synchronous to async/await"
        exit 1
        ;;
esac

echo "Running migration: $MIGRATION_TYPE"
echo "Source directory: $SOURCE_DIR"
echo ""

BACKUP_DIR="backup_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

find "$SOURCE_DIR" -name "$FILE_PATTERN" -type f | while read file; do
    echo "Migrating: $file"

    # Backup original
    cp "$file" "$BACKUP_DIR/"

    # Perform migration
    MIGRATED=$(claude --print "$PROMPT

\`\`\`
$(cat "$file")
\`\`\`

Return only the migrated code, no explanations.")

    # Determine output file
    if [ "$NEW_EXT" != "${file##*.}" ]; then
        new_file="${file%.*}$NEW_EXT"
    else
        new_file="$file"
    fi

    echo "$MIGRATED" > "$new_file"

    # Remove old file if extension changed
    if [ "$new_file" != "$file" ]; then
        rm "$file"
    fi

    echo "  -> $new_file"
done

echo ""
echo "Migration complete!"
echo "Backups saved to: $BACKUP_DIR/"
echo ""
echo "Review changes with: git diff"
```

### Example 7: Log Analysis Pipeline

```bash
#!/bin/bash
# analyze_logs.sh - AI-powered log analysis

LOG_FILE="${1:-/var/log/app.log}"
ANALYSIS_DEPTH="${2:-100}"  # Number of recent lines to analyze

if [ ! -f "$LOG_FILE" ]; then
    echo "Log file not found: $LOG_FILE"
    exit 1
fi

OUTPUT_DIR="log_analysis_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "Analyzing: $LOG_FILE"
echo "Last $ANALYSIS_DEPTH lines"
echo ""

# Extract recent logs
RECENT_LOGS=$(tail -n "$ANALYSIS_DEPTH" "$LOG_FILE")

# Step 1: Error Detection
echo "Step 1: Detecting errors..."
ERRORS=$(claude --print "Extract all error messages from these logs:

$RECENT_LOGS

Return as JSON:
{
  \"errors\": [
    {\"timestamp\": \"...\", \"level\": \"ERROR\", \"message\": \"...\", \"count\": N}
  ],
  \"total_errors\": N
}" --output-format json)

echo "$ERRORS" > "$OUTPUT_DIR/errors.json"
echo "  Errors saved to errors.json"

# Step 2: Pattern Analysis
echo "Step 2: Analyzing patterns..."
PATTERNS=$(claude --print "Analyze patterns in these logs:

$RECENT_LOGS

Identify:
1. Recurring error patterns
2. Unusual activity
3. Performance issues
4. Security concerns

Format as a report.")

echo "$PATTERNS" > "$OUTPUT_DIR/patterns.md"
echo "  Patterns saved to patterns.md"

# Step 3: Recommendations
echo "Step 3: Generating recommendations..."
RECOMMENDATIONS=$(claude --print "Based on this log analysis:

Errors:
$(cat "$OUTPUT_DIR/errors.json")

Patterns:
$(cat "$OUTPUT_DIR/patterns.md")

Provide actionable recommendations:
1. Immediate actions needed
2. Monitoring suggestions
3. Code fixes required
4. Infrastructure changes

Prioritize by severity.")

echo "$RECOMMENDATIONS" > "$OUTPUT_DIR/recommendations.md"
echo "  Recommendations saved to recommendations.md"

# Step 4: Summary Dashboard
echo "Step 4: Creating summary..."
SUMMARY=$(claude --print "Create a brief executive summary of this log analysis:

$(cat "$OUTPUT_DIR/recommendations.md")

Include:
- Overall health status (Good/Warning/Critical)
- Top 3 issues
- Recommended immediate action

Keep under 200 words.")

echo "$SUMMARY" > "$OUTPUT_DIR/summary.md"

# Display summary
echo ""
echo "=========================================="
echo "LOG ANALYSIS SUMMARY"
echo "=========================================="
cat "$OUTPUT_DIR/summary.md"
echo "=========================================="
echo ""
echo "Full analysis saved to: $OUTPUT_DIR/"
```

---

## Output Format Comparison

Understanding when to use each output format is crucial for effective automation.

### Format Overview Table

| Format | Use Case | Parsing | Human Readable |
|--------|----------|---------|----------------|
| `text` | Scripts, display, simple capture | Basic (grep, awk) | Excellent |
| `json` | APIs, data processing, complex parsing | jq, programming languages | Poor |
| `markdown` | Documentation, reports, formatted output | Basic or render | Good |

### Text Format Details

```bash
# Default text output
claude --print "Explain variables"

# Characteristics:
# - Plain text, no special formatting
# - Good for direct display
# - Easy to pipe to other commands
# - Human-friendly

# Best practices:
# - Use for simple Q&A
# - Use when output will be displayed directly
# - Use when piping to text-processing tools

# Example usage:
claude --print "What is Python?" | head -5
claude --print "List 5 items" | wc -l
```

### JSON Format Details

```bash
# JSON output
claude --print "List 3 colors" --output-format json

# Expected output:
# {"colors": ["red", "blue", "green"]}

# Characteristics:
# - Structured data
# - Machine-parseable
# - Type-safe when parsed
# - Consistent format

# Best practices:
# - Use for data extraction
# - Use when integrating with APIs
# - Use when you need specific fields
# - Always validate JSON output

# Example usage:
RESULT=$(claude --print "Return JSON with name and age" --output-format json)
NAME=$(echo "$RESULT" | jq -r '.name')
AGE=$(echo "$RESULT" | jq -r '.age')

# Handling JSON arrays
ITEMS=$(claude --print "List 5 items as JSON array" --output-format json)
echo "$ITEMS" | jq -r '.[]'
```

### Markdown Format Details

```bash
# Markdown output
claude --print "Explain OOP" --output-format markdown

# Expected output:
# # Object-Oriented Programming
#
# **OOP** is a programming paradigm...
#
# ## Key Concepts
# - Encapsulation
# - Inheritance
# - Polymorphism

# Characteristics:
# - Formatted text
# - Headers, lists, code blocks
# - Can be rendered
# - Good for documentation

# Best practices:
# - Use for documentation generation
# - Use for reports
# - Use when output will be rendered
# - Save directly to .md files

# Example usage:
claude --print "Document this function" --output-format markdown > docs/function.md
```

### Format Conversion

```bash
# Convert between formats
# JSON to text
claude --print "..." --output-format json | jq -r '.message'

# Text to JSON (wrap in JSON)
TEXT=$(claude --print "...")
echo "{\"response\": \"$TEXT\"}" | jq .

# Markdown to HTML (using pandoc)
claude --print "..." --output-format markdown | pandoc -f markdown -t html
```

---

## Exit Codes Reference

Understanding exit codes is essential for robust scripting.

### Exit Code Table

| Code | Name | Meaning | Common Causes |
|------|------|---------|---------------|
| 0 | Success | Request completed successfully | Normal operation |
| 1 | General Error | Unspecified error occurred | Various issues |
| 2 | Invalid Arguments | Bad command-line arguments | Typos, missing required args |
| 3 | API Error | Problem with Anthropic API | Auth, rate limits, server issues |
| 124 | Timeout | Command timed out | Long prompts, slow network |

### Handling Exit Codes

```bash
#!/bin/bash
# Comprehensive exit code handling

handle_claude_result() {
    local exit_code=$1
    local output="$2"

    case $exit_code in
        0)
            echo "Success"
            echo "$output"
            return 0
            ;;
        1)
            echo "Error: General failure"
            echo "Output: $output"
            return 1
            ;;
        2)
            echo "Error: Invalid arguments"
            echo "Check your command syntax"
            echo "Details: $output"
            return 2
            ;;
        3)
            echo "Error: API issue"
            if echo "$output" | grep -q "rate limit"; then
                echo "Rate limited - waiting 60 seconds..."
                sleep 60
                return 3  # Caller can retry
            elif echo "$output" | grep -q "auth"; then
                echo "Authentication failed - check ANTHROPIC_API_KEY"
                return 3
            else
                echo "API error: $output"
                return 3
            fi
            ;;
        124)
            echo "Error: Request timed out"
            echo "Try a shorter prompt or increase timeout"
            return 124
            ;;
        *)
            echo "Error: Unknown exit code $exit_code"
            echo "Output: $output"
            return $exit_code
            ;;
    esac
}

# Usage
output=$(claude --print "Hello" 2>&1)
exit_code=$?
handle_claude_result $exit_code "$output"
```

### Exit Code Best Practices

```bash
#!/bin/bash
# Best practices for exit code handling

set -e  # Exit on error (use with caution)

# Method 1: Direct check
if claude --print "Test" > /dev/null 2>&1; then
    echo "Claude is available"
else
    echo "Claude is not available"
    exit 1
fi

# Method 2: Capture and check
result=$(claude --print "Test" 2>&1) || {
    echo "Claude failed: $result"
    exit 1
}

# Method 3: Explicit code check
claude --print "Test"
code=$?
[ $code -eq 0 ] || exit $code

# Method 4: Retry on specific codes
retry_claude() {
    local max_attempts=3
    local attempt=1

    while [ $attempt -le $max_attempts ]; do
        if result=$(claude --print "$1" 2>&1); then
            echo "$result"
            return 0
        fi

        code=$?
        if [ $code -eq 3 ]; then
            echo "API error, attempt $attempt/$max_attempts" >&2
            sleep $((attempt * 10))
            attempt=$((attempt + 1))
        else
            return $code
        fi
    done

    return 1
}
```

---

## Troubleshooting

### "API key not found" in Scripts

**Problem**: Script fails with API key error even though interactive mode works.

**Causes and Solutions**:

```bash
# Cause 1: Environment variable not exported
# Wrong:
ANTHROPIC_API_KEY="YOUR_API_KEY"
claude --print "test"

# Correct:
export ANTHROPIC_API_KEY="YOUR_API_KEY"
claude --print "test"

# Cause 2: Different shell environment
# When running from cron or systemd, environment differs

# Solution: Set in script explicitly
#!/bin/bash
export ANTHROPIC_API_KEY="YOUR_API_KEY"
# Or source from file
source ~/.claude_env

# Cause 3: Subshell doesn't inherit
# Wrong (in some cases):
(claude --print "test")

# Solution: Pass explicitly
ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" claude --print "test"

# Cause 4: Systemd service
# Add to service file:
# [Service]
# Environment="ANTHROPIC_API_KEY=YOUR_API_KEY"
```

### Output Truncation

**Problem**: Long responses are cut off.

**Solutions**:

```bash
# Solution 1: Request concise output
claude --print "Summarize briefly: $(cat large_file.txt)"

# Solution 2: Process in chunks
split_and_process() {
    local file="$1"
    local chunk_size=1000

    split -l $chunk_size "$file" /tmp/chunk_

    for chunk in /tmp/chunk_*; do
        claude --print "Process this chunk: $(cat "$chunk")"
    done

    rm /tmp/chunk_*
}

# Solution 3: Use streaming (if available)
claude --print "Long analysis" --stream

# Solution 4: Request specific format
claude --print "List only the top 10 items from: ..."
```

### Timeout Issues

**Problem**: Requests timeout on complex prompts.

**Solutions**:

```bash
# Solution 1: Use system timeout
timeout 300 claude --print "Complex analysis..."

# Solution 2: Simplify prompt
# Instead of:
claude --print "Analyze everything about this 10000 line file"

# Use:
claude --print "List the main functions in this file"

# Solution 3: Background processing
claude --print "Long task" &
pid=$!
sleep 5
while kill -0 $pid 2>/dev/null; do
    echo "Still processing..."
    sleep 10
done
wait $pid

# Solution 4: Break into steps
STEP1=$(claude --print "Step 1: Extract main points")
STEP2=$(claude --print "Step 2: Analyze: $STEP1")
STEP3=$(claude --print "Step 3: Summarize: $STEP2")
```

### Encoding Problems

**Problem**: Special characters cause issues.

**Solutions**:

```bash
# Solution 1: Use base64 encoding
ENCODED=$(cat file.txt | base64)
claude --print "Decode and analyze this base64: $ENCODED"

# Solution 2: Use heredoc with quoting
claude --print "$(cat <<'EOF'
Analyze this text with special chars: "quotes" 'apostrophes' $dollars
EOF
)"

# Solution 3: Escape special characters
TEXT=$(cat file.txt | sed 's/"/\\"/g' | sed "s/'/\\'/g")
claude --print "Analyze: $TEXT"

# Solution 4: Use temporary file
cat file.txt > /tmp/input.txt
claude --print "Analyze the file at /tmp/input.txt: $(cat /tmp/input.txt)"

# Solution 5: Set locale
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
claude --print "Text with émojis and üñíçödé"
```

### Exit Code Handling Issues

**Problem**: Scripts don't properly catch errors.

**Solutions**:

```bash
# Solution 1: Always capture stderr
result=$(claude --print "test" 2>&1)
code=$?

# Solution 2: Use set -e carefully
set -e
set -o pipefail  # Catch errors in pipes

# Solution 3: Trap errors
trap 'echo "Error on line $LINENO"; exit 1' ERR

# Solution 4: Explicit checking
if ! result=$(claude --print "test" 2>&1); then
    echo "Failed: $result" >&2
    exit 1
fi

# Solution 5: Wrapper function
safe_claude() {
    local result
    local code

    result=$(claude "$@" 2>&1)
    code=$?

    if [ $code -ne 0 ]; then
        echo "Claude error ($code): $result" >&2
        return $code
    fi

    echo "$result"
}
```

### JSON Parsing Failures

**Problem**: JSON output is invalid or parsing fails.

**Solutions**:

```bash
# Solution 1: Validate JSON
RESPONSE=$(claude --print "Return JSON" --output-format json)
if echo "$RESPONSE" | jq . > /dev/null 2>&1; then
    echo "Valid JSON"
else
    echo "Invalid JSON: $RESPONSE"
fi

# Solution 2: Extract JSON from mixed output
RESPONSE=$(claude --print "Return JSON")
JSON=$(echo "$RESPONSE" | grep -o '{.*}' | head -1)

# Solution 3: Request strict JSON
claude --print "Return ONLY valid JSON, no explanation:
{\"key\": \"value\"}" --output-format json

# Solution 4: Use jq with error handling
if ! VALUE=$(echo "$RESPONSE" | jq -r '.key' 2>/dev/null); then
    echo "Failed to parse JSON"
    exit 1
fi

# Solution 5: Default values
VALUE=$(echo "$RESPONSE" | jq -r '.key // "default"')
```

---

## Best Practices

### 1. Always Handle Errors

```bash
#!/bin/bash
# GOOD: Comprehensive error handling

set -euo pipefail

claude_with_retry() {
    local prompt="$1"
    local max_retries="${2:-3}"
    local retry_delay="${3:-5}"

    for ((i=1; i<=max_retries; i++)); do
        if result=$(claude --print "$prompt" 2>&1); then
            echo "$result"
            return 0
        fi

        echo "Attempt $i failed, retrying in ${retry_delay}s..." >&2
        sleep "$retry_delay"
    done

    echo "All $max_retries attempts failed" >&2
    return 1
}

# BAD: No error handling
# result=$(claude --print "test")
```

### 2. Use Appropriate Output Format

```bash
# GOOD: Match format to use case

# For human display
claude --print "Explain this concept"

# For data extraction
claude --print "Extract data" --output-format json | jq '.data'

# For documentation
claude --print "Document this" --output-format markdown > doc.md

# BAD: Using JSON for display
# claude --print "Tell me a story" --output-format json
```

### 3. Set Timeouts for Long Operations

```bash
# GOOD: Explicit timeouts

# System timeout
timeout 120 claude --print "Long analysis"

# Or wrapper with timeout
timed_claude() {
    timeout "${CLAUDE_TIMEOUT:-60}" claude "$@"
}

# BAD: No timeout (can hang indefinitely)
# claude --print "Analyze this massive codebase"
```

### 4. Log Responses for Debugging

```bash
# GOOD: Comprehensive logging

LOG_DIR="${LOG_DIR:-/var/log/claude}"
mkdir -p "$LOG_DIR"

logged_claude() {
    local prompt="$1"
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local log_file="$LOG_DIR/claude_$timestamp.log"

    {
        echo "=== Request ==="
        echo "Time: $(date)"
        echo "Prompt: $prompt"
        echo ""
        echo "=== Response ==="
    } > "$log_file"

    if result=$(claude --print "$prompt" 2>&1); then
        echo "$result" >> "$log_file"
        echo "$result"
    else
        echo "ERROR: $result" >> "$log_file"
        return 1
    fi
}

# BAD: No logging
# result=$(claude --print "important task")
```

### 5. Validate Inputs

```bash
# GOOD: Input validation

process_file() {
    local file="$1"

    # Validate file exists
    if [ ! -f "$file" ]; then
        echo "Error: File not found: $file" >&2
        return 1
    fi

    # Validate file size
    local size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file")
    if [ "$size" -gt 100000 ]; then
        echo "Error: File too large (max 100KB)" >&2
        return 1
    fi

    # Validate content
    if ! file "$file" | grep -q "text"; then
        echo "Error: Not a text file" >&2
        return 1
    fi

    claude --print "Process: $(cat "$file")"
}

# BAD: No validation
# claude --print "Process: $(cat "$1")"
```

### 6. Use Consistent Prompt Structure

```bash
# GOOD: Structured prompts

CODE_REVIEW_PROMPT='You are a senior code reviewer.

## Task
Review the following code for bugs, security issues, and improvements.

## Code
```
%s
```

## Required Output Format
1. **Issues Found**: List any bugs or problems
2. **Security Concerns**: Note any security issues
3. **Suggestions**: Recommend improvements
4. **Rating**: Score from 1-10'

review_code() {
    local code="$1"
    local prompt=$(printf "$CODE_REVIEW_PROMPT" "$code")
    claude --print "$prompt"
}

# BAD: Unstructured prompts
# claude --print "review this code $code"
```

### 7. Handle Rate Limits Gracefully

```bash
# GOOD: Rate limit handling

LAST_CALL=0
MIN_INTERVAL=1  # Minimum seconds between calls

rate_limited_claude() {
    local now=$(date +%s)
    local elapsed=$((now - LAST_CALL))

    if [ $elapsed -lt $MIN_INTERVAL ]; then
        sleep $((MIN_INTERVAL - elapsed))
    fi

    LAST_CALL=$(date +%s)
    claude "$@"
}

# For batch processing
batch_with_rate_limit() {
    local files=("$@")
    local delay=2

    for file in "${files[@]}"; do
        claude --print "Process: $(cat "$file")"
        sleep $delay
    done
}
```

---

## Quick Reference Cheat Sheet

### Basic Commands

```bash
# Simple prompt
claude --print "Your question here"
claude -p "Your question here"  # Short form

# With output format
claude --print "Question" --output-format json
claude --print "Question" --output-format markdown
claude --print "Question" --output-format text

# With model selection
claude --print "Question" --model opus
claude --print "Question" --model sonnet
```

### Input Methods

```bash
# Direct string
claude --print "What is 2+2?"

# Variable
MSG="Explain this"
claude --print "$MSG"

# Command substitution
claude --print "Review: $(cat file.txt)"

# Pipe
cat file.txt | claude --print "Summarize this"
echo "Hello" | claude --print "Translate to French"

# Heredoc
claude --print "$(cat <<'EOF'
Multi-line
prompt here
EOF
)"
```

### Output Handling

```bash
# Capture to variable
RESULT=$(claude --print "Question")

# Save to file
claude --print "Question" > output.txt

# Display and save
claude --print "Question" | tee output.txt

# Parse JSON
claude --print "Q" --output-format json | jq '.key'

# Filter output
claude --print "List items" | grep "pattern"
claude --print "Count things" | head -10
```

### Error Handling

```bash
# Check success
if claude --print "Test" > /dev/null 2>&1; then
    echo "Success"
fi

# Capture exit code
claude --print "Test"
echo "Exit code: $?"

# Capture output and code
result=$(claude --print "Test" 2>&1)
code=$?

# Retry pattern
for i in 1 2 3; do
    claude --print "Test" && break
    sleep 5
done
```

### Common Patterns

```bash
# Code review
claude --print "Review: $(cat code.py)"

# Documentation
claude --print "Document: $(cat module.py)" --output-format markdown

# Translation
claude --print "Translate to Spanish: Hello world"

# JSON extraction
claude --print "Return JSON: {\"key\": \"value\"}" --output-format json

# Batch processing
for f in *.py; do
    claude --print "Analyze: $(cat "$f")" > "$f.analysis"
done
```

### Environment Variables

```bash
# Required
export ANTHROPIC_API_KEY="YOUR_API_KEY"

# Optional (if supported)
export CLAUDE_MODEL="opus"
export CLAUDE_TIMEOUT="120"
```

### Quick Templates

```bash
# Template: Basic question
claude --print "What is [TOPIC]?"

# Template: Code analysis
claude --print "Analyze this code for [ASPECT]:
\`\`\`
$(cat [FILE])
\`\`\`"

# Template: Format conversion
cat [INPUT] | claude --print "Convert to [FORMAT]"

# Template: Summarization
claude --print "Summarize in [N] sentences: $(cat [FILE])"

# Template: JSON output
claude --print "[QUESTION]. Return as JSON." --output-format json
```

---

## Complete Script Templates

### Template: Production-Ready Wrapper

```bash
#!/bin/bash
# claude_wrapper.sh - Production-ready Claude Code wrapper
# Usage: source claude_wrapper.sh

# Configuration
CLAUDE_LOG_DIR="${CLAUDE_LOG_DIR:-/tmp/claude_logs}"
CLAUDE_MAX_RETRIES="${CLAUDE_MAX_RETRIES:-3}"
CLAUDE_RETRY_DELAY="${CLAUDE_RETRY_DELAY:-5}"
CLAUDE_TIMEOUT="${CLAUDE_TIMEOUT:-120}"

mkdir -p "$CLAUDE_LOG_DIR"

# Logging function
_claude_log() {
    local level="$1"
    local message="$2"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message" >> "$CLAUDE_LOG_DIR/claude.log"
}

# Main wrapper function
claude_safe() {
    local prompt="$1"
    local output_format="${2:-text}"
    local retries=0
    local result
    local exit_code

    # Validate
    if [ -z "$prompt" ]; then
        _claude_log "ERROR" "Empty prompt provided"
        echo "Error: Prompt required" >&2
        return 2
    fi

    if [ -z "$ANTHROPIC_API_KEY" ]; then
        _claude_log "ERROR" "API key not set"
        echo "Error: ANTHROPIC_API_KEY not set" >&2
        return 3
    fi

    # Execute with retries
    while [ $retries -lt $CLAUDE_MAX_RETRIES ]; do
        _claude_log "INFO" "Attempt $((retries + 1)): ${prompt:0:50}..."

        result=$(timeout "$CLAUDE_TIMEOUT" claude --print "$prompt" \
            --output-format "$output_format" 2>&1)
        exit_code=$?

        if [ $exit_code -eq 0 ]; then
            _claude_log "INFO" "Success"
            echo "$result"
            return 0
        elif [ $exit_code -eq 124 ]; then
            _claude_log "WARN" "Timeout"
            retries=$((retries + 1))
        elif [ $exit_code -eq 3 ]; then
            _claude_log "WARN" "API error: $result"
            retries=$((retries + 1))
            sleep "$CLAUDE_RETRY_DELAY"
        else
            _claude_log "ERROR" "Failed with code $exit_code: $result"
            echo "$result" >&2
            return $exit_code
        fi
    done

    _claude_log "ERROR" "All retries exhausted"
    echo "Error: All retries failed" >&2
    return 1
}

# JSON-specific function
claude_json() {
    local prompt="$1"
    local result

    result=$(claude_safe "$prompt" json)
    local code=$?

    if [ $code -ne 0 ]; then
        return $code
    fi

    # Validate JSON
    if echo "$result" | jq . > /dev/null 2>&1; then
        echo "$result"
    else
        _claude_log "ERROR" "Invalid JSON response"
        echo "Error: Invalid JSON response" >&2
        return 1
    fi
}

# Export functions
export -f claude_safe
export -f claude_json
export -f _claude_log
```

### Template: CI/CD Integration Script

```bash
#!/bin/bash
# ci_claude_review.sh - Complete CI/CD code review script

set -euo pipefail

# Configuration
REVIEW_OUTPUT="code_review_report.md"
MAX_FILE_SIZE=50000  # 50KB max per file
SUPPORTED_EXTENSIONS="js|ts|py|go|java|rb|rs"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1"; }

# Check prerequisites
check_prerequisites() {
    if ! command -v claude &> /dev/null; then
        log_error "Claude Code not installed"
        exit 1
    fi

    if [ -z "${ANTHROPIC_API_KEY:-}" ]; then
        log_error "ANTHROPIC_API_KEY not set"
        exit 1
    fi

    if ! command -v jq &> /dev/null; then
        log_warn "jq not installed, JSON parsing limited"
    fi
}

# Get changed files
get_changed_files() {
    local base_branch="${1:-main}"
    git diff --name-only "$base_branch"...HEAD | grep -E "\.($SUPPORTED_EXTENSIONS)$" || true
}

# Review a single file
review_file() {
    local file="$1"
    local diff

    # Check file size
    if [ ! -f "$file" ]; then
        log_warn "File not found: $file"
        return
    fi

    local size=$(wc -c < "$file")
    if [ "$size" -gt "$MAX_FILE_SIZE" ]; then
        log_warn "File too large, skipping: $file"
        return
    fi

    # Get diff
    diff=$(git diff main...HEAD -- "$file" 2>/dev/null || cat "$file")

    # Request review
    log_info "Reviewing: $file"

    local review
    review=$(claude --print "Review this code change concisely:

File: $file

\`\`\`diff
$diff
\`\`\`

Provide:
1. Summary (1 line)
2. Issues (if any)
3. Suggestions (if any)

Be brief." 2>&1) || {
        log_error "Failed to review $file"
        return
    }

    echo "$review"
}

# Main function
main() {
    log_info "Starting code review"

    check_prerequisites

    local base_branch="${1:-main}"
    local changed_files
    changed_files=$(get_changed_files "$base_branch")

    if [ -z "$changed_files" ]; then
        log_info "No files to review"
        exit 0
    fi

    # Initialize report
    {
        echo "# Code Review Report"
        echo ""
        echo "Generated: $(date)"
        echo "Base branch: $base_branch"
        echo ""
        echo "## Files Reviewed"
        echo ""
    } > "$REVIEW_OUTPUT"

    # Review each file
    local file_count=0
    local issue_count=0

    for file in $changed_files; do
        file_count=$((file_count + 1))

        echo "### $file" >> "$REVIEW_OUTPUT"
        echo "" >> "$REVIEW_OUTPUT"

        review=$(review_file "$file")
        echo "$review" >> "$REVIEW_OUTPUT"
        echo "" >> "$REVIEW_OUTPUT"

        # Count issues (simple heuristic)
        if echo "$review" | grep -qi "issue\|bug\|error\|problem"; then
            issue_count=$((issue_count + 1))
        fi
    done

    # Summary
    {
        echo "## Summary"
        echo ""
        echo "- Files reviewed: $file_count"
        echo "- Files with issues: $issue_count"
        echo ""
    } >> "$REVIEW_OUTPUT"

    log_info "Review complete: $REVIEW_OUTPUT"
    log_info "Files reviewed: $file_count, Issues found: $issue_count"

    # Exit with error if issues found (for CI)
    if [ "$issue_count" -gt 0 ]; then
        exit 1
    fi
}

main "$@"
```

### Template: Batch Processor with Progress

```bash
#!/bin/bash
# batch_processor.sh - Process multiple files with progress tracking

set -euo pipefail

# Configuration
PARALLEL_JOBS="${PARALLEL_JOBS:-4}"
OUTPUT_DIR="${OUTPUT_DIR:-./processed}"
LOG_FILE="${LOG_FILE:-batch_process.log}"

# Progress tracking
TOTAL_FILES=0
PROCESSED_FILES=0
FAILED_FILES=0
START_TIME=$(date +%s)

# Initialize
init() {
    mkdir -p "$OUTPUT_DIR"
    echo "Batch Processing Started: $(date)" > "$LOG_FILE"
}

# Progress display
show_progress() {
    local current=$1
    local total=$2
    local failed=$3
    local elapsed=$(($(date +%s) - START_TIME))
    local rate=0

    if [ $elapsed -gt 0 ]; then
        rate=$((current * 60 / elapsed))
    fi

    local pct=$((current * 100 / total))
    local bar_width=40
    local filled=$((pct * bar_width / 100))
    local empty=$((bar_width - filled))

    printf "\r["
    printf "%${filled}s" | tr ' ' '='
    printf "%${empty}s" | tr ' ' ' '
    printf "] %3d%% (%d/%d) Failed: %d Rate: %d/min" \
        $pct $current $total $failed $rate
}

# Process single file
process_file() {
    local file="$1"
    local operation="$2"
    local output_file="$OUTPUT_DIR/$(basename "$file").processed"

    # Read file content
    local content
    content=$(cat "$file")

    # Process with Claude
    local result
    if result=$(claude --print "$operation

\`\`\`
$content
\`\`\`

Return only the processed result." 2>&1); then
        echo "$result" > "$output_file"
        echo "[SUCCESS] $file -> $output_file" >> "$LOG_FILE"
        return 0
    else
        echo "[FAILED] $file: $result" >> "$LOG_FILE"
        return 1
    fi
}

# Main batch processor
batch_process() {
    local pattern="$1"
    local operation="$2"

    # Find all files
    local files=()
    while IFS= read -r -d '' file; do
        files+=("$file")
    done < <(find . -name "$pattern" -type f -print0)

    TOTAL_FILES=${#files[@]}

    if [ $TOTAL_FILES -eq 0 ]; then
        echo "No files matching pattern: $pattern"
        exit 0
    fi

    echo "Processing $TOTAL_FILES files with pattern: $pattern"
    echo "Operation: $operation"
    echo "Output directory: $OUTPUT_DIR"
    echo ""

    # Process files
    for file in "${files[@]}"; do
        if process_file "$file" "$operation"; then
            PROCESSED_FILES=$((PROCESSED_FILES + 1))
        else
            FAILED_FILES=$((FAILED_FILES + 1))
        fi

        show_progress $((PROCESSED_FILES + FAILED_FILES)) $TOTAL_FILES $FAILED_FILES

        # Rate limiting
        sleep 1
    done

    echo ""
    echo ""
    echo "=========================================="
    echo "Batch Processing Complete"
    echo "=========================================="
    echo "Total files: $TOTAL_FILES"
    echo "Processed: $PROCESSED_FILES"
    echo "Failed: $FAILED_FILES"
    echo "Time: $(($(date +%s) - START_TIME)) seconds"
    echo "Log: $LOG_FILE"
    echo "Output: $OUTPUT_DIR/"
    echo "=========================================="
}

# Usage
usage() {
    echo "Usage: $0 <file-pattern> <operation>"
    echo ""
    echo "Examples:"
    echo "  $0 '*.py' 'Add docstrings to all functions'"
    echo "  $0 '*.js' 'Convert to TypeScript'"
    echo "  $0 '*.md' 'Fix grammar and spelling'"
}

# Main
if [ $# -lt 2 ]; then
    usage
    exit 1
fi

init
batch_process "$1" "$2"
```

---

## Conclusion

Headless mode transforms Claude Code from an interactive assistant into a powerful
automation tool. By mastering the concepts in this tutorial, you can:

- **Automate repetitive tasks** with shell scripts
- **Integrate AI into CI/CD pipelines** for automated code review
- **Process files in batch** with parallel execution
- **Build data pipelines** that leverage AI analysis
- **Create scheduled jobs** for ongoing analysis

Remember the key principles:
1. Always handle errors gracefully
2. Use the appropriate output format for your use case
3. Set timeouts to prevent hanging
4. Log responses for debugging
5. Validate inputs before processing

Start with simple scripts and gradually build more complex automations as you
become comfortable with headless mode. The examples and templates in this tutorial
provide a solid foundation for production-ready Claude Code automation.

---

## Additional Resources

- Claude Code Documentation: https://docs.anthropic.com/claude-code
- Bash Scripting Guide: https://tldp.org/LDP/abs/html/
- jq Manual: https://stedolan.github.io/jq/manual/
- GNU Parallel Tutorial: https://www.gnu.org/software/parallel/parallel_tutorial.html

---

*Tutorial version 1.0 - Last updated: 2025*
