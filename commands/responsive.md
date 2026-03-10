---
description: Make screens responsive across phone and tablet sizes — adds responsive breakpoints, adaptive layouts, and responsive spacing/font scaling.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path or screen_file>"
---

# Make Screen Responsive

Add responsive layout support to screens for phone and tablet sizes.

## Arguments

- `$ARGUMENTS`: Feature path or specific screen file.

## Step 1: Check for flutter_screenutil

Check if the project uses `flutter_screenutil` or has responsive helpers:
```bash
grep -r "screenutil\|ScreenUtil\|responsive" lib/utils/
```

## Step 2: Responsive Patterns

### Using MediaQuery (built-in)
```dart
@override
Widget build(BuildContext context) {
  final size = MediaQuery.sizeOf(context);
  final isTablet = size.width > 600;
  final isLandscape = size.width > size.height;

  return Scaffold(
    body: isTablet
        ? _buildTabletLayout(context)
        : _buildPhoneLayout(context),
  );
}
```

### Using LayoutBuilder (for parent-aware sizing)
```dart
LayoutBuilder(
  builder: (context, constraints) {
    final crossAxisCount = constraints.maxWidth > 600 ? 3 : 2;
    return GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: crossAxisCount,
      ),
      // ...
    );
  },
)
```

### Responsive Grid
```dart
Widget _buildGrid(BuildContext context, List<ItemModel> items) {
  final width = MediaQuery.sizeOf(context).width;
  final crossAxisCount = width > 900 ? 4 : width > 600 ? 3 : 2;
  final childAspectRatio = width > 600 ? 0.8 : 0.7;

  return GridView.builder(
    gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: crossAxisCount,
      childAspectRatio: childAspectRatio,
      crossAxisSpacing: 12,
      mainAxisSpacing: 12,
    ),
    itemCount: items.length,
    itemBuilder: (context, index) => ItemCard(item: items[index]),
  );
}
```

### Responsive Padding
```dart
EdgeInsets _responsivePadding(BuildContext context) {
  final width = MediaQuery.sizeOf(context).width;
  if (width > 900) return const EdgeInsets.symmetric(horizontal: 64, vertical: 24);
  if (width > 600) return const EdgeInsets.symmetric(horizontal: 32, vertical: 16);
  return const EdgeInsets.all(16);
}
```

### Responsive Text
```dart
double _responsiveFontSize(BuildContext context, double baseSize) {
  final width = MediaQuery.sizeOf(context).width;
  if (width > 600) return baseSize * 1.2;
  return baseSize;
}
```

## Step 3: Tablet Layout Patterns

### Side-by-side (Master-Detail)
```dart
if (isTablet) {
  return Row(
    children: [
      SizedBox(width: 320, child: MasterList()),
      const VerticalDivider(width: 1),
      Expanded(child: DetailView()),
    ],
  );
}
```

### Constrained width for forms
```dart
Center(
  child: ConstrainedBox(
    constraints: const BoxConstraints(maxWidth: 600),
    child: FormContent(),
  ),
)
```

## Rules

- Use `MediaQuery.sizeOf(context)` (not `.of(context).size`) for performance.
- Prefer `LayoutBuilder` when sizing depends on parent constraints.
- Don't hardcode pixel breakpoints — use consistent constants.
- Test on both phone (375px) and tablet (768px+) widths.
- Keep phone layout as default, add tablet as enhancement.
