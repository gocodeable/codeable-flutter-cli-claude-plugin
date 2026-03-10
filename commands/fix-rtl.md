---
description: Migrate a feature or the entire project to support RTL (Right-to-Left) layouts — replaces directional padding/margin/alignment with Directional variants.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "[feature-path] or leave empty for full project"
---

# Fix RTL Support

Migrate Flutter code to support RTL layouts by replacing directional-aware properties with their `Directional` counterparts.

## Arguments

- `$ARGUMENTS`: Optional feature path. If empty, scan the entire `lib/` directory.

## Replacements

### EdgeInsets → EdgeInsetsDirectional

```dart
// Before
EdgeInsets.only(left: 16, right: 8)
padding: const EdgeInsets.only(left: 16)
margin: EdgeInsets.only(right: 12)

// After
EdgeInsetsDirectional.only(start: 16, end: 8)
padding: const EdgeInsetsDirectional.only(start: 16)
margin: EdgeInsetsDirectional.only(end: 12)
```

Mapping:
- `left:` → `start:`
- `right:` → `end:`
- `top:` and `bottom:` stay the same
- `EdgeInsets.symmetric()` is already RTL-safe — skip
- `EdgeInsets.all()` is already RTL-safe — skip
- `EdgeInsets.fromLTRB(l, t, r, b)` → `EdgeInsetsDirectional.fromSTEB(s, t, e, b)`

### Alignment → AlignmentDirectional

```dart
// Before
Alignment.centerLeft → AlignmentDirectional.centerStart
Alignment.centerRight → AlignmentDirectional.centerEnd
Alignment.topLeft → AlignmentDirectional.topStart
Alignment.topRight → AlignmentDirectional.topEnd
Alignment.bottomLeft → AlignmentDirectional.bottomStart
Alignment.bottomRight → AlignmentDirectional.bottomEnd
```

Skip: `Alignment.center`, `Alignment.topCenter`, `Alignment.bottomCenter` (already RTL-safe).

### Positioned → PositionedDirectional

```dart
// Before
Positioned(left: 0, ...)
Positioned(right: 0, ...)

// After
PositionedDirectional(start: 0, ...)
PositionedDirectional(end: 0, ...)
```

### Container → Container with Directional

When a `Container` uses `alignment` or `padding`/`margin` with directional values, update those properties.

### Border/BorderRadius

```dart
// Before
BorderRadius.only(topLeft: ..., bottomRight: ...)

// After
BorderRadiusDirectional.only(topStart: ..., bottomEnd: ...)
```

## Process

1. Glob all `.dart` files in the target path.
2. For each file, search for non-RTL-safe patterns.
3. Replace with RTL-safe equivalents.
4. Preserve `const` where possible.
5. Run `dart analyze lib/` after all changes.
6. Fix any issues and re-verify.

## Rules

- Don't change `EdgeInsets.symmetric()` or `EdgeInsets.all()` — they're already RTL-safe.
- Don't change `Alignment.center` / `topCenter` / `bottomCenter` — they're already RTL-safe.
- Preserve existing formatting and code style.
- Don't make any other code changes beyond RTL fixes.
