---
description: Localize all hardcoded UI strings in a Flutter feature directory using context.l10n pattern. Recursively scans, adds ARB keys, updates the localization service, replaces strings in code, and verifies.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "<feature-path> e.g. lib/features/customer/customer_home"
---

# Localize Feature

You are a localization agent for codeable_cli Flutter projects. Find and replace ALL hardcoded user-facing strings with `context.l10n.xxx` calls.

## Arguments

- `$ARGUMENTS`: The feature directory path to localize. If not provided, ask the user.

## Key Files

Detect these by scanning the project:
- **English ARB**: `lib/l10n/arb/app_en.arb`
- **Other ARBs**: `lib/l10n/arb/app_*.arb`
- **l10n extension**: `lib/l10n/l10n.dart` (provides `context.l10n`)
- **Localization service**: `lib/l10n/localization_service.dart` (static fallback)

## Phase 1: Discovery

1. Glob for all `.dart` files in the feature directory.
2. Read each file and identify hardcoded user-facing strings:
   - `Text('...')`, button labels, AppBar titles, hint text, error messages, dialog content, tooltips, tab labels, snackbar messages, TextSpan content
3. SKIP: route paths, asset paths, API keys/URLs, map keys, enum values, color hex codes, font names, log messages, empty strings, formatting characters.
4. Build manifest: `{file, line, original_string, proposed_key}`.

## Phase 2: Key Naming

- **camelCase** always
- Prefix with semantic context: `orderDetailsTitle`, `buttonSubmit`, `errorNoConnection`
- Reuse existing keys if the exact same English string exists in ARB
- For parameterized strings, use ICU format: `"priceLabel": "Just {price}"`

## Phase 3: Update ARB Files

1. Add new keys to ALL ARB files.
2. Provide translations for non-English ARBs.
3. Include `@key` metadata for parameterized messages.
4. Don't remove existing keys. Maintain valid JSON.

## Phase 4: Update Localization Service

If `localization_service.dart` exists, add static getters for new keys:
```dart
static String get keyName => _instance.keyName;
static String priceLabel(String price) => _instance.priceLabel(price);
```

## Phase 5: Replace Strings in Code

**In widgets/screens (have BuildContext):**
- Use `context.l10n.keyName`
- Import `package:{pkg}/l10n/l10n.dart`
- Handle `const` removal where needed

**In utility files (no BuildContext):**
- Use `Localization.keyName`
- Import `package:{pkg}/l10n/localization_service.dart`

## Phase 6: Verify

1. Run `flutter gen-l10n` to regenerate.
2. Run `dart analyze lib/` and fix errors.
3. Repeat until clean.

## Phase 7: Recursive Check

Re-scan for remaining hardcoded strings. Loop back if any found. Only complete when zero remain.

## Rules

- Always prefer `context.l10n.xxx` over `Localization.xxx`.
- Only use `Localization.xxx` in files without BuildContext.
- One widget class per file.
- Don't refactor or improve code beyond localization.
- If `const` context contains `context.l10n`, remove the `const`.
