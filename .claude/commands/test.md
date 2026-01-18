# Run Tests

Intelligently run tests based on project type and changes.

## Instructions

1. Detect the project type and test framework:
   - Check for `package.json` → look for test script and framework (jest, vitest, mocha, etc.)
   - Check for `pytest.ini`, `pyproject.toml` → pytest
   - Check for `Cargo.toml` → cargo test
   - Check for `go.mod` → go test
   - Check for `Makefile` → look for test target
2. If arguments provided, run specific tests
3. If no arguments:
   - Check `git diff --name-only` for changed files
   - Determine related test files
   - Run focused tests if changes are limited, full suite otherwise
4. Execute tests and report results
5. If tests fail:
   - Show the failing tests clearly
   - Analyze the failure
   - Suggest fixes if obvious

## Common Test Commands

| Framework | Command |
|-----------|---------|
| Jest | `npm test` or `npx jest` |
| Vitest | `npm test` or `npx vitest` |
| Pytest | `pytest` or `python -m pytest` |
| Go | `go test ./...` |
| Cargo | `cargo test` |
| PHPUnit | `./vendor/bin/phpunit` |

## Arguments

$ARGUMENTS - Optional: specific test file, pattern, or flags

## Examples

- `/test` - Run all tests or tests related to changes
- `/test auth` - Run tests matching "auth"
- `/test --coverage` - Run with coverage
- `/test src/utils/date.test.ts` - Run specific test file
- `/test --watch` - Run in watch mode
