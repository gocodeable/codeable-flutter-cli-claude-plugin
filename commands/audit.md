---
description: Audit the project structure against codeable_cli conventions — checks for missing layers, naming violations, cubit/state file counts, import patterns, and architectural drift.
allowed-tools: Read, Bash, Glob, Grep
argument-hint: "(no arguments needed)"
---

# Project Architecture Audit

Audit a codeable_cli-generated Flutter project for convention violations and architectural drift.

## Checks

### 1. Feature Structure Integrity

For each feature directory under `lib/features/`:
- Verify `data/repositories/` exists with `*_repository_impl.dart`
- Verify `domain/repositories/` exists with `*_repository.dart`
- Verify `presentation/cubit/` exists with exactly `cubit.dart` and `state.dart`
- Verify `presentation/views/` exists with at least one screen
- Flag any extra cubit/state files (violation: ONE cubit per feature)
- Flag missing layers

### 2. Naming Convention Violations

- Cubit files must be named `cubit.dart` and `state.dart` (not `feature_cubit.dart`)
- Repository interfaces: `{prefix}_repository.dart`
- Repository impls: `{prefix}_repository_impl.dart`
- Screen files: `{prefix}_screen.dart` or `{prefix}_{name}_screen.dart`
- Widget files: `{prefix}_{purpose}.dart`

### 3. Widget File Discipline

- Flag any file with more than one `StatelessWidget`/`StatefulWidget` class
- Flag screens with excessively long build methods (>100 lines) that should be extracted

### 4. Import Patterns

- Flag files not using the barrel export (`exports.dart`)
- Flag files importing from `core/` or `utils/` individually when barrel covers it
- Flag cross-feature imports (feature A importing from feature B's internals)

### 5. State Management

- Verify all DataState fields have defaults of `const DataState.initial()`
- Verify all state classes extend Equatable
- Verify `props` includes all fields
- Verify `copyWith` covers all fields
- Flag any state field NOT wrapped in DataState (for async data)

### 6. Repository Patterns

- Flag any repository impl that injects ApiService via constructor (should be singleton)
- Flag any bare `Exception` catch (should be `AppApiException`)
- Verify all repo methods return `RepositoryResponse<T>`

### 7. DI & Registration

- Verify every feature cubit is registered in `app_page.dart`
- Verify every feature has a route in `router.dart`
- Flag orphaned registrations (registered cubit but feature deleted)

### 8. Route Integrity

- Verify every route has a corresponding screen file
- Verify every screen is exported in `exports.dart`
- Flag routes pointing to non-existent screens

## Output

Generate a report:

```
## Architecture Audit Report

### ✅ Passing Checks
- [list of passing checks]

### ⚠️ Warnings
- [list of non-critical issues]

### ❌ Violations
- [list of convention violations with file paths and descriptions]

### 📊 Summary
- Features: X total, Y fully compliant, Z with issues
- Total violations: N
- Total warnings: N
```

## Rules

- This is READ-ONLY — do not modify any files.
- Report findings clearly with file paths and line numbers.
- Prioritize violations by severity (structural > naming > style).
