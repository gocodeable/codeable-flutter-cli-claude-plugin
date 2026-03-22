---
name: coding-standards
description: Comprehensive coding standards and quality rules for codeable_cli Flutter projects — code cleanliness, dead code removal, analyzer hints, commit conventions, feature extraction criteria, parallelization, and strict quality gates. Use alongside codeable-architecture, codeable-conventions, and error-handling skills.
---

# Coding Standards & Quality Rules

Strict quality rules for codeable_cli Flutter projects. These complement the architecture, conventions, and error-handling skills with enforcement rules.

## Code Cleanliness

### No Dead Code

- **Delete unused imports, variables, methods, classes, widgets, and models** — do not leave them in the codebase
- **No commented-out code** — if code is removed, delete it entirely. Git history preserves it if needed
- **No commented-out files** — if a file is no longer used, delete it
- **No unused widgets or widget files** — if a widget is no longer referenced, delete the file
- **No unused models** — if a model is no longer used anywhere, delete it and its folder

### Resolve All Analyzer Hints

- **Resolve all analyzer hints and warnings** — do not suppress them with `// ignore:` unless absolutely necessary
- Fix the underlying issue rather than ignoring the lint
- If a hint suggests a better pattern (e.g., `const` constructor, removing unnecessary `this`, using collection literals), adopt it
- Run `dart analyze` and ensure zero issues before considering work complete

### Comments

- **No useless comments** — don't restate what the code does
- **No doc comments on self-explanatory methods/classes** — `/// Gets the items` on a method called `getItems()` adds nothing
- **No section separators** like `// ===== Section =====` or `// --- Section ---`
- **Only comment where the "why" isn't obvious** from the code itself
- Remove `print` / `debugPrint` statements — use `AppLogger` only where appropriate

## Git Commits

- **No `Co-Authored-By: Claude` or any AI co-author lines** in commit messages
- Write concise commit messages that describe what changed and why
- Use conventional commit prefixes: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`, `style:`

## Feature Extraction Criteria

A concern gets its own feature folder when:
- It has its own cubit/state (e.g., active orders, events, notifications)
- It has its own repository and API endpoints
- It has its own screens/views
- It would bloat the parent feature's cubit with unrelated state

If a feature is growing too large (cubit has 30+ methods, state has 20+ fields), evaluate which concerns can be extracted into separate features.

## View Rules

### No `setState()`

Never use `setState()` — use Cubit state management exclusively. If you need reactive UI updates that don't warrant cubit state, use `ValueNotifier` with `ValueListenableBuilder`.

### No `_build` Methods

Never write private `_buildXyz()` methods in views. If a section of UI is complex enough to be a method, extract it into its own `StatelessWidget` file in `presentation/widgets/`.

### No Private Widgets in View Files

Do not define private widget classes (`_MyWidget`) in view files. Extract them into their own `StatelessWidget` files in `presentation/widgets/`.

### No Business Logic in Views

Views call cubit methods — they do not contain API calls, link generation, data processing, or service interactions. All business logic belongs in the cubit.

### View File Size Limit

If your view file exceeds ~1000 lines, you are not extracting enough widgets. Refactor immediately.

### Cache Cubit References

When using multiple cubits in a method, cache them in local variables:

```dart
Future<void> _initialize() async {
  final homeCubit = context.read<CustomerHomeCubit>();
  final ordersCubit = context.read<CustomerActiveOrdersCubit>();
  final eventsCubit = context.read<CustomerEventsCubit>();
  // use cached refs instead of calling context.read repeatedly
}
```

### Dispose Controllers

Always dispose `TextEditingController`, `ScrollController`, `FocusNode`, `ValueNotifier`, and any other disposable resources in `dispose()`.

## Parallelization

Use `Future.wait` for independent async calls — never await them sequentially:

```dart
// CORRECT
await Future.wait([
  cubitA.fetchData(),
  cubitB.fetchData(),
  cubitC.fetchData(),
]);

// WRONG — sequential when they could run in parallel
await cubitA.fetchData();
await cubitB.fetchData();
await cubitC.fetchData();
```

## BlocBuilder / BlocListener Precision

- **Always use `buildWhen`** on `BlocBuilder` to limit rebuilds to only the state fields that matter
- **Always use `listenWhen`** on `BlocListener` to limit side effects to relevant state changes
- **Extract listener logic** into named methods when the listener body is complex
- **If your `listenWhen` checks 5+ variables, split the listener** — use `MultiBlocListener` or multiple `BlocListener` widgets with narrow, focused `listenWhen` clauses

```dart
BlocListener<MyCubit, MyState>(
  listenWhen: (prev, curr) => prev.deleteState != curr.deleteState,
  listener: _handleDeleteState,
  child: ...
)
```

## Use Project Custom Components

Never use raw Flutter widgets when project-level alternatives exist:

| Instead of | Use |
|---|---|
| `AlertDialog`, `showDialog` with `TextButton` | `CustomDialog` / `MueblyDialog` |
| `ElevatedButton`, `TextButton` | Project button (`RMBButton` / `MueblyButton`) |
| `AppBar` | Project `appBar()` function |
| `print()`, `debugPrint()` | `AppLogger` |
| `SnackBar` | `ToastHelper` |
| Manual date string manipulation | `DateFormat` from `intl` + `DateTimeHelper` |

Check `utils/widgets/core_widgets/` for the project's actual widget names.

## Repository Error Handling

All repository methods use `execute()` for centralized error handling — no manual try-catch:

```dart
@override
Future<RepositoryResponse<List<ItemModel>>> getItems() {
  return execute(() async {
    final response = await _apiService.get(Endpoints.items);
    final data = response.data['data'] as List<dynamic>;
    return data.map((e) => ItemModel.fromJson(e as Map<String, dynamic>)).toList();
  });
}
```

- `execute()` is defined in `repository_response.dart` alongside `RepositoryResponse`
- Catches `AppApiException` (returns error message) and unexpected errors (logs with `AppLogger.error` + stack trace, returns "Something went wrong")
- Throw `AppApiException` inside the callback for business-level failures
- No manual `AppLogger.error()` calls needed in repositories — `execute()` handles logging

## Hive Models

- Use `@HiveType` / `@HiveField` annotations (or `@GenerateAdapters` for Hive CE) with code generation
- **No manual Hive adapters** — always use generated `.g.dart` files via `build_runner`
- Run `dart run build_runner build` after adding/modifying Hive-annotated models

## Models

### One Model Per File

Never put multiple model classes in a single file. Each model gets its own `.dart` file in its own folder. Creating files is free — untangling models from shared files is not.

### Pagination

For paginated features, **always use `PaginationModel<T>`** — never define separate `currentPage`, `totalPages`, `hasMore`, and `List<T> items` fields in state.

### Model Resilience

- `fromJson` factories must handle null/missing fields gracefully
- Use `as Type? ?? defaultValue` for safe casting
- **List parsing must skip malformed items** rather than crashing:

```dart
final items = (json['items'] as List<dynamic>?)
    ?.map((e) {
      try {
        return ItemModel.fromJson(e as Map<String, dynamic>);
      } catch (_) {
        return null;
      }
    })
    .whereType<ItemModel>()
    .toList() ?? [];
```

## Socket / Real-time

Socket connections are initialized from cubits (e.g., `initializeSocketConnection()`), not from views. Socket event handlers update cubit state.

## Image Picking

Store picked images in cubit state as `XFile?` fields from `image_picker`, not raw file paths.

## Permission Handling

Use `PermissionManager` with proper callbacks:

```dart
final hasPermission = await PermissionManager.requestLocationPermission(
  onGranted: () {},
  onDenied: (message) {
    AppLogger.error('Permission denied: $message');
    ToastHelper.showErrorToast('Permission required');
  },
  onPermanentlyDenied: () {
    CustomDialog.showOpenSettingsDialog(
      context: context,
      title: 'Please enable permission in Settings.',
    );
  },
);
```

## Enums & Extensions

- Create enums for any finite set of values (statuses, types, roles)
- Create **enum extensions** for display names, colors, and icons rather than scattering switch statements across the codebase
- Enum files go in `data/models/` or `core/` if shared

## Naming Conventions

- **No spelling mistakes** in class names, variable names, file names, or widget names
- Widget folders organized by concern using subfolders, not flat dumps of 20 files
- All names should be descriptive and self-documenting

## Reusability

- Reusability is the top priority when writing widgets and utilities
- Before creating a new widget, check if a similar one already exists
- Common UI patterns should be abstracted into shared widgets in `utils/widgets/`

## AI-Generated Code

- **Do not blindly accept AI-generated code** — review every file it modifies
- If the AI generates `setState`, raw `AlertDialog`, generic `catch (e)`, or manual Hive adapters, fix it before committing
- AI tools don't know project conventions unless told — enforcement is your responsibility

## Quick Reference Checklist

When writing or reviewing code, verify:

- [ ] No dead code, commented code, unused files/widgets/models
- [ ] All analyzer hints and warnings resolved (not ignored)
- [ ] No useless comments or section separators
- [ ] No `setState()` — cubit state only
- [ ] No `_build` methods — widgets extracted to separate files
- [ ] No private widgets in view files
- [ ] No business logic in views — delegate to cubit
- [ ] View files under ~1000 lines
- [ ] `buildWhen` / `listenWhen` on all `BlocBuilder` / `BlocListener`
- [ ] `listenWhen` clauses narrow and focused (not 5+ variables)
- [ ] Independent async calls use `Future.wait`
- [ ] Custom UI components used (not raw Flutter dialogs/buttons)
- [ ] Repository methods use `execute()` — no manual try-catch
- [ ] Hive models use code generation (no manual adapters)
- [ ] One model per file — no multi-model files
- [ ] `PaginationModel<T>` for paginated state
- [ ] Models parse resiliently with null safety
- [ ] Enums with extensions for display names, colors, icons
- [ ] No spelling mistakes in names
- [ ] Widget folders organized by concern (subfolders)
- [ ] AI-generated code reviewed before committing
- [ ] No `Co-Authored-By` in commit messages
- [ ] `dart analyze` passes with zero issues
