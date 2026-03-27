---
description: Create and wire a bottom sheet for a feature — action sheets, confirmation sheets, form sheets, or custom content sheets using the project's CustomBottomSheet core widget.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<feature_path> <sheet_name> [--type action|confirmation|form|custom]"
---

# Add Bottom Sheet

Create a bottom sheet widget for a feature and wire it to the calling screen.

## Arguments

Parse `$ARGUMENTS` for:
- **Feature path** (required): e.g., `lib/features/customer/customer_orders`
- **Sheet name** (required): e.g., `order_actions`, `delete_confirm`, `edit_address`
- **Type** (optional): `--type action|confirmation|form|custom` (default: `action`)

If arguments are missing, ask the user.

## Step 1: Detect Configuration

1. Read `pubspec.yaml` for package name.
2. List the feature's existing files to understand the structure.
3. Read the feature's existing cubit and state files.
4. Read `lib/utils/widgets/core_widgets/bottom_sheet.dart` to understand the `CustomBottomSheet` API.
5. Read existing screens in the feature to understand where the sheet will be triggered.

## Step 2: Create Bottom Sheet Widget

Create `presentation/widgets/{prefix}_{sheet_name}_sheet.dart` in the feature directory.

### Action Sheet (list of actions)
```dart
import 'package:{pkg}/exports.dart';

class {Prefix}{SheetName}Sheet extends StatelessWidget {
  const {Prefix}{SheetName}Sheet({
    super.key,
    required this.onSelected,
  });

  final void Function(String action) onSelected;

  static void show(
    BuildContext context, {
    required void Function(String action) onSelected,
  }) {
    CustomBottomSheet.show(
      context: context,
      title: '{Sheet Title}',
      body: {Prefix}{SheetName}Sheet(onSelected: onSelected),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        ListTile(
          leading: const Icon(Icons.edit_outlined),
          title: Text('Edit', style: context.b1),
          onTap: () {
            Navigator.pop(context);
            onSelected('edit');
          },
        ),
        ListTile(
          leading: Icon(Icons.delete_outline, color: AppColors.error),
          title: Text(
            'Delete',
            style: context.b1.copyWith(color: AppColors.error),
          ),
          onTap: () {
            Navigator.pop(context);
            onSelected('delete');
          },
        ),
      ],
    );
  }
}
```

### Confirmation Sheet (confirm/cancel)
```dart
import 'package:{pkg}/exports.dart';

class {Prefix}{SheetName}Sheet extends StatelessWidget {
  const {Prefix}{SheetName}Sheet({
    super.key,
    required this.onConfirm,
    this.isDestructive = false,
  });

  final VoidCallback onConfirm;
  final bool isDestructive;

  static void show(
    BuildContext context, {
    required VoidCallback onConfirm,
    bool isDestructive = false,
  }) {
    CustomBottomSheet.show(
      context: context,
      title: '{Confirmation Title}',
      subtitle: '{Confirmation message}',
      buttonOneText: 'Confirm',
      buttonOneOnTap: () {
        Navigator.pop(context);
        onConfirm();
      },
      buttonOneColor: isDestructive ? AppColors.error : null,
      buttonTwoText: 'Cancel',
      buttonTwoOnTap: () => Navigator.pop(context),
    );
  }

  @override
  Widget build(BuildContext context) {
    return const SizedBox.shrink();
  }
}
```

Note: For simple confirmation sheets, prefer using `CustomBottomSheet.show(...)` directly from the calling screen instead of creating a wrapper widget. Only create a separate widget when the sheet has custom body content beyond title/subtitle/buttons.

### Form Sheet (with inputs)
```dart
import 'package:{pkg}/exports.dart';

class {Prefix}{SheetName}Sheet extends StatefulWidget {
  const {Prefix}{SheetName}Sheet({
    super.key,
    required this.onSubmit,
  });

  final void Function({/* params */}) onSubmit;

  static void show(
    BuildContext context, {
    required void Function({/* params */}) onSubmit,
  }) {
    CustomBottomSheet.show(
      context: context,
      title: '{Form Title}',
      height: 0.6,
      body: {Prefix}{SheetName}Sheet(onSubmit: onSubmit),
    );
  }

  @override
  State<{Prefix}{SheetName}Sheet> createState() =>
      _{Prefix}{SheetName}SheetState();
}

class _{Prefix}{SheetName}SheetState extends State<{Prefix}{SheetName}Sheet> {
  final _formKey = GlobalKey<FormState>();
  late final TextEditingController _controller;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          CustomTextField(
            controller: _controller,
            hintText: '{Field hint}',
          ),
          const SizedBox(height: 16),
          CustomButton(
            text: 'Submit',
            onTap: () {
              if (_formKey.currentState?.validate() ?? false) {
                Navigator.pop(context);
                widget.onSubmit(/* values */);
              }
            },
          ),
        ],
      ),
    );
  }
}
```

### Custom Content Sheet
```dart
import 'package:{pkg}/exports.dart';

class {Prefix}{SheetName}Sheet extends StatelessWidget {
  const {Prefix}{SheetName}Sheet({super.key});

  static void show(BuildContext context) {
    CustomBottomSheet.show(
      context: context,
      title: '{Sheet Title}',
      body: const {Prefix}{SheetName}Sheet(),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        // Custom content here
      ],
    );
  }
}
```

## Step 3: Wire to Calling Screen

1. Read the screen where the bottom sheet will be triggered.
2. Add the import for the new sheet widget.
3. Wire the `show()` call to the appropriate trigger (button tap, long press, menu icon, etc.).

Example wiring:
```dart
// In the screen's build method or action handler:
IconButton(
  icon: const Icon(Icons.more_vert),
  onPressed: () => {Prefix}{SheetName}Sheet.show(
    context,
    onSelected: (action) {
      if (action == 'edit') cubit.startEdit();
      if (action == 'delete') cubit.deleteItem(id: item.id);
    },
  ),
),
```

If the sheet triggers cubit actions:
- Ensure the cubit methods exist — add them if they don't.
- Use `context.read<{Prefix}Cubit>()` to access the cubit from the sheet if needed.

## Step 4: Verify

1. Run `dart analyze lib/` and fix any issues.
2. Report the file path and how to trigger the sheet.

## Rules

- Always use the project's `CustomBottomSheet` core widget — never use raw `showModalBottomSheet`.
- Add a static `show()` method on the widget for convenience.
- Pop the sheet before executing action callbacks to avoid stale context.
- One widget class per file.
- Use `StatelessWidget` unless the sheet has form controllers or animation controllers.
- For simple confirmation with just title/subtitle/buttons, call `CustomBottomSheet.show(...)` directly — no wrapper widget needed.
- Business logic belongs in the cubit, not the sheet — the sheet only calls cubit methods.
- Match the naming convention of existing widgets in the feature.
- No useless comments.
