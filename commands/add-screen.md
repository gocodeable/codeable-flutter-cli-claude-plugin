---
description: Add a new screen to an existing feature — creates the view file, wires the route, and adds navigation. No new cubit — uses the feature's existing cubit.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<screen_name> --feature <feature_path>"
---

# Add Screen to Existing Feature

Add a new screen/view to an existing feature without creating a new cubit. The screen shares the feature's existing cubit.

## Arguments

Parse `$ARGUMENTS` for:
- **Screen name** (required): e.g., `order_details`, `edit_profile`
- **Feature path** (optional): `--feature lib/features/customer/customer_orders`. If not provided, ask.

## Step 1: Detect Configuration

1. Read `pubspec.yaml` for package name.
2. List the feature's existing files to understand the structure.
3. Read the feature's existing cubit and state files.
4. Read `lib/go_router/routes.dart` and `router.dart` for routing patterns.

## Step 2: Create Screen File

Create `presentation/views/{prefix}_{screen_name}_screen.dart`:

```dart
import 'package:{pkg}/exports.dart';
import 'package:{pkg}/features/{path}/presentation/cubit/cubit.dart';
import 'package:{pkg}/features/{path}/presentation/cubit/state.dart';

class {Prefix}{ScreenName}Screen extends StatelessWidget {
  const {Prefix}{ScreenName}Screen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: mueblyAppBar(context: context, title: '{Screen Title}'),
      body: BlocBuilder<{Prefix}Cubit, {Prefix}State>(
        builder: (context, state) {
          return const Center(
            child: Text('{Screen Title}'),
          );
        },
      ),
    );
  }
}
```

Note: Always use StatelessWidget for screens unless you have actual mutable state (TextEditingControllers, AnimationControllers). BlocBuilder/BlocListener do NOT require StatefulWidget.
```

If the screen needs a route parameter (like an ID), add it:
```dart
class {Prefix}{ScreenName}Screen extends StatelessWidget {
  const {Prefix}{ScreenName}Screen({super.key, required this.id});
  final String id;
  // ...
}
```

## Step 3: Wire Route

1. Add to `lib/go_router/routes.dart`:
```dart
static const String {prefix}{ScreenName}Screen = '/{role}-{feature}-{screen}';
```

2. Add to `lib/go_router/router.dart` following existing pattern.

3. Add export to `lib/go_router/exports.dart`.

4. Add route name to `AppRouteNames` if used.

## Step 4: Verify

1. Run `dart analyze lib/` and fix any issues.
2. Report the route path for navigation.

## Rules

- Do NOT create a new cubit — use the feature's existing one.
- Match the appBar widget used by the project (check existing screens).
- Follow the exact file naming convention of existing screens.
- If the screen needs data, add state fields and cubit methods to the EXISTING cubit/state.
- No useless comments — don't add `/// Widget that shows...`, `// Title`, or section separators. Only comment non-obvious logic.
- Business logic belongs in the cubit, not the UI — time calculations, event payloads, refresh orchestration go in cubit methods. The screen only calls cubit methods.
