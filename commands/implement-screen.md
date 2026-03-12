---
description: Build a complete screen from a description or Figma reference — uses existing core widgets, AppColors, text styles, and handles loading/error/empty states.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<feature-path> e.g. lib/features/profile"
---

# Implement Screen

Build a complete screen using the project's core widgets, theme system, and architecture patterns.

## Arguments

Parse $ARGUMENTS for the feature path. Then ask:
- Do you have a **description** of the screen, or a **Figma URL**?

## Step 1: Read Available Resources

1. Read core widgets directory to understand available widgets (CustomAppBar, CustomButton, CustomTextField, PaginatedListView, etc.)
2. Read `lib/constants/app_colors.dart` for colors
3. Read text style extensions: `context.h1`, `context.t1`, `context.b1`, `context.l1`
4. Read `lib/constants/asset_paths.dart` for assets
5. Read the feature's cubit and state for available data

## Step 2: Build the Screen

Create/update the screen file using:
- `BlocBuilder` for UI rendering, `BlocConsumer` when side effects needed
- `CustomAppBar` for app bars
- `context.l10n.keyName` for all user-facing strings
- `EdgeInsetsDirectional` for RTL support
- `AppColors` for all colors (never hardcoded)
- `context.h1/t1/b1/l1` for text styles (never raw TextStyle)

## Step 3: Handle All States

- **Loading**: CircularProgressIndicator or shimmer
- **Failure**: Error message with retry button
- **Empty**: Empty state illustration
- **Loaded**: Actual content

## Step 4: Extract Widgets

If any section is complex (>40 lines), extract to `<feature>/presentation/widgets/`.

## Step 5: Verify

Run `dart analyze --no-pub` and fix issues.

## Rules

- ALWAYS use existing core widgets before building custom ones
- ALWAYS use context.l10n for strings, AppColors for colors, context.h1/b1 for styles
- ALWAYS use EdgeInsetsDirectional (never EdgeInsets.left/right)
- Handle all states (loading, error, empty, loaded)
