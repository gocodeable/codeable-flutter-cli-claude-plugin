---
description: Create a reusable dialog or bottom sheet widget for a feature — confirmation dialogs, action sheets, form dialogs, or info dialogs.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> <dialog_name> [--type confirmation|form|info|action-sheet]"
---

# Add Dialog/Bottom Sheet

Create a dialog or bottom sheet widget for a feature.

## Arguments

- `$ARGUMENTS`: Feature path, dialog name, and type.

## Dialog Types

### Confirmation Dialog
```dart
class {Prefix}ConfirmDialog extends StatelessWidget {
  const {Prefix}ConfirmDialog({
    super.key,
    required this.title,
    required this.message,
    required this.onConfirm,
    this.confirmText = 'Confirm',
    this.cancelText = 'Cancel',
    this.isDestructive = false,
  });

  final String title;
  final String message;
  final VoidCallback onConfirm;
  final String confirmText;
  final String cancelText;
  final bool isDestructive;

  static Future<void> show(BuildContext context, {...}) {
    return showDialog(
      context: context,
      builder: (_) => {Prefix}ConfirmDialog(...),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MueblyDialog(
      title: title,
      content: Text(message, style: context.b1),
      actions: [
        MueblyTextButton(
          text: cancelText,
          onPressed: () => Navigator.pop(context),
        ),
        MueblyButton(
          text: confirmText,
          backgroundColor: isDestructive ? AppColors.error : null,
          onPressed: () {
            Navigator.pop(context);
            onConfirm();
          },
        ),
      ],
    );
  }
}
```

### Action Sheet (Bottom Sheet)
```dart
class {Prefix}ActionSheet extends StatelessWidget {
  const {Prefix}ActionSheet({super.key, required this.onSelected});

  final void Function(String action) onSelected;

  static Future<void> show(BuildContext context, {...}) {
    return showModalBottomSheet(
      context: context,
      builder: (_) => {Prefix}ActionSheet(...),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MueblyBottomSheet(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          ListTile(
            leading: const Icon(Icons.edit),
            title: Text('Edit', style: context.b1),
            onTap: () {
              Navigator.pop(context);
              onSelected('edit');
            },
          ),
          ListTile(
            leading: Icon(Icons.delete, color: AppColors.error),
            title: Text('Delete', style: context.b1.copyWith(color: AppColors.error)),
            onTap: () {
              Navigator.pop(context);
              onSelected('delete');
            },
          ),
        ],
      ),
    );
  }
}
```

### Form Dialog
Uses `StatefulWidget` with controllers and Form key.

## Placement

Widget goes in `presentation/widgets/{prefix}_{dialog_name}.dart`.

## Usage

```dart
// Confirmation
{Prefix}ConfirmDialog.show(
  context,
  title: 'Delete Item',
  message: 'Are you sure?',
  isDestructive: true,
  onConfirm: () => cubit.deleteItem(id: item.id),
);

// Action sheet
{Prefix}ActionSheet.show(
  context,
  onSelected: (action) {
    if (action == 'edit') context.push(AppRoutes.editItem, extra: item);
    if (action == 'delete') cubit.deleteItem(id: item.id);
  },
);
```

## Rules

- Use core `MueblyDialog` and `MueblyBottomSheet` widgets.
- Add static `show()` method for convenience.
- Pop the dialog before executing the action callback.
- One widget class per file.
