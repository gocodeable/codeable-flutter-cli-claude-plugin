---
description: Extract a section of a screen's build method into a separate widget file in the feature's widgets/ folder. Maintains proper imports and passes required data via constructor.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<screen_file_path> [widget_name]"
---

# Extract Widget

Extract a section of a screen's build method into a separate widget file in the feature's `presentation/widgets/` folder.

## Arguments

- `$ARGUMENTS`: Path to the screen file, and optionally the widget name to create. If no widget name, ask the user which section to extract.

## Step 1: Read the Screen

Read the screen file and present the build method structure to the user. Ask which section(s) to extract if not obvious.

## Step 2: Analyze Dependencies

For the section to extract, identify:
- **State fields** accessed (e.g., `state.someData`)
- **Cubit methods** called (e.g., `context.read<Cubit>().method()`)
- **Local variables** from the parent scope
- **BuildContext** usage

Determine if the widget should:
- **Accept data via constructor** (for pure display widgets)
- **Use BlocBuilder internally** (for widgets that need reactive state)
- **Accept callbacks** (for widgets that trigger cubit actions)

## Step 3: Create Widget File

Create `presentation/widgets/{prefix}_{widget_name}.dart`:

```dart
import 'package:{pkg}/exports.dart';

class {Prefix}{WidgetName} extends StatelessWidget {
  const {Prefix}{WidgetName}({
    super.key,
    required this.data,
    this.onAction,
  });

  final DataType data;
  final VoidCallback? onAction;

  @override
  Widget build(BuildContext context) {
    // Extracted widget tree
  }
}
```

## Step 4: Update Screen

Replace the extracted section with the new widget:
```dart
{Prefix}{WidgetName}(
  data: state.someData.data!,
  onAction: () => context.read<Cubit>().someMethod(),
),
```

## Step 5: Verify

Run `dart analyze lib/` and fix any issues.

## Rules

- ONE widget class per file.
- Widget file goes in the feature's `presentation/widgets/` folder.
- Follow the project's naming convention: `{prefix}_{purpose}.dart`.
- Use `const` constructors where possible.
- Prefer passing data via constructor over accessing cubit directly in the widget.
- Keep the extracted widget as simple and focused as possible.
