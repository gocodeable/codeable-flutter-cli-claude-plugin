---
description: Review recently changed code for reuse opportunities, code quality issues, and efficiency — then fix any issues found. Checks for duplicate logic, missing const, over-engineering, and convention violations.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "(no arguments needed)"
---

# Simplify Changed Code

Review all recently changed files for quality issues and fix them.

## Step 1: Identify Changed Files

Run `git diff --name-only HEAD` and `git diff --name-only --cached` to find modified files.

## Step 2: Review Each File

For each changed `.dart` file, check:

### Reuse Opportunities
- Is there duplicate logic that exists elsewhere in the project?
- Could an existing core widget be used instead of custom code?
- Is there a utility/helper that already does what the new code does?

### Code Quality
- Missing `const` constructors or `const` widget instantiation
- Unnecessary null checks on non-nullable types
- Over-complicated null handling (e.g., `x ?? ''` when x is already non-null)
- Dead code or unreachable branches
- Unnecessary type annotations (Dart infers them)

### Convention Violations
- Multiple widget classes in one file
- Missing Equatable props
- State fields without DataState wrapper
- Wrong import pattern (not using barrel export)
- Repository catching bare Exception instead of AppApiException

### Over-Engineering
- Unnecessary abstractions for one-time logic
- Extra error handling for impossible scenarios
- Feature flags or backward compatibility for code that can just change
- Helper functions used only once

### Efficiency
- N+1 patterns in list building
- Unnecessary rebuilds (BlocBuilder wrapping too much)
- Missing `buildWhen` / `listenWhen` for targeted rebuilds

## Step 3: Fix Issues

Apply fixes directly. For each fix, keep changes minimal and focused.

## Step 4: Verify

Run `dart analyze lib/` and ensure no new issues introduced.
