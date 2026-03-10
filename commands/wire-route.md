---
description: Wire a new route for an existing screen — adds route path constant, route name, GoRoute entry, and screen export.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<screen_file_path> [route_path]"
---

# Wire Route

Add routing for an existing screen that doesn't have a route yet.

## Arguments

- `$ARGUMENTS`: Path to the screen file and optionally the desired route path.

## Steps

1. Read `pubspec.yaml` for package name.
2. Read the screen file to get the class name and any required parameters.
3. Read `lib/go_router/routes.dart` to understand the route constants pattern.
4. Read `lib/go_router/router.dart` to understand the routing structure.
5. Read `lib/go_router/exports.dart` for the barrel export pattern.

### Add Route Path

Add to `routes.dart`:
```dart
static const String {prefix}Screen = '/{route-path}';
```

### Add Route Name (if AppRouteNames exists)

```dart
static const String {prefix}Screen = '{prefix}-screen';
```

### Add GoRoute

Add to `router.dart` following the existing pattern:
```dart
GoRoute(
  path: AppRoutes.{prefix}Screen,
  name: AppRouteNames.{prefix}Screen,
  builder: (context, state) => const {Prefix}Screen(),
),
```

If the screen has route parameters:
```dart
GoRoute(
  path: '${AppRoutes.{prefix}Screen}/:id',
  name: AppRouteNames.{prefix}Screen,
  builder: (context, state) => {Prefix}Screen(
    id: state.pathParameters['id']!,
  ),
),
```

### Add Screen Export

Add to `exports.dart`:
```dart
export 'package:{pkg}/features/{path}/presentation/views/{screen_file}.dart';
```

### Verify

Run `dart analyze lib/` and fix any issues.
