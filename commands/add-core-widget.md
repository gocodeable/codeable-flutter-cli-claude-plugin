---
description: Create a new reusable core widget in utils/widgets/core_widgets/ — follows the project's widget conventions, adds to barrel export, and includes all customization props.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<widget_name> e.g. StarRating, ExpandableCard, CountdownTimer"
---

# Add Core Widget

Create a new reusable widget in `utils/widgets/core_widgets/` following project conventions.

## Arguments

- `$ARGUMENTS`: Widget name (PascalCase). If not provided, ask what widget to create.

## Step 1: Check Existing Widgets

Read `lib/utils/widgets/core_widgets/export.dart` to see naming pattern and existing widgets.

## Step 2: Determine Widget Prefix

Check existing widgets for the project's prefix convention (e.g., `Muebly`, `Custom`, or no prefix).

## Step 3: Create Widget File

`lib/utils/widgets/core_widgets/{snake_case_name}.dart`:

```dart
import 'package:{pkg}/exports.dart';

class {Prefix}{WidgetName} extends StatelessWidget {
  const {Prefix}{WidgetName}({
    super.key,
    // Required props first
    required this.requiredProp,
    // Optional props with defaults
    this.optionalProp = defaultValue,
    this.onAction,
  });

  final RequiredType requiredProp;
  final OptionalType optionalProp;
  final VoidCallback? onAction;

  @override
  Widget build(BuildContext context) {
    return // widget tree
  }
}
```

## Step 4: Add to Barrel Export

Add to `lib/utils/widgets/core_widgets/export.dart`:
```dart
export '{snake_case_name}.dart';
```

## Step 5: Verify

Run `dart analyze lib/` and fix any issues.

## Widget Design Principles

- Use `const` constructor
- Accept all customization via constructor (colors, sizes, callbacks)
- Provide sensible defaults for optional props
- Use `AppColors` and `context.xxx` text styles — don't hardcode values
- Support `disabled` state where applicable
- One public widget class per file — no private `_build` helper methods; extract sub-sections into their own widget files in `presentation/widgets/`
- Keep the widget focused on one responsibility
- Use StatelessWidget by default — only use StatefulWidget when you have actual mutable state (TextEditingControllers, AnimationControllers, etc.). BlocBuilder/BlocListener do NOT require StatefulWidget
- No useless comments — don't add comments that restate what the code does (no `/// Widget that shows...`, no `// Title`, no section separators). Only comment non-obvious logic

## Common Widget Patterns

### With Loading State
```dart
final bool isLoading;
// Show CircularProgressIndicator or shimmer when loading
```

### With Error State
```dart
final String? errorText;
// Show error message below/around the widget
```

### With Callback
```dart
final ValueChanged<T>? onChanged;
final VoidCallback? onPressed;
```

### With Theme Variants
```dart
enum WidgetVariant { primary, secondary, outlined }
final WidgetVariant variant;
```
