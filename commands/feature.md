---
description: Scaffold a new feature with full Clean Architecture layers — data/domain/presentation, cubit, state, repository, route, and DI wiring. Like `codeable_cli feature` but AI-powered with smarter defaults.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_name> [--role <role>]"
---

# Generate Feature

You are a feature scaffolding agent for codeable_cli Flutter projects. Generate a complete feature module with all layers wired together.

## Arguments

Parse `$ARGUMENTS` for:
- **Feature name** (required): snake_case name (e.g., `orders`, `product_details`)
- **Role** (optional): `--role customer`, `--role admin`, `--role business`. If not provided, check if the project uses roles by looking at `lib/features/` subdirectories.

If no feature name is provided, ask the user.

## Step 1: Detect Project Configuration

1. Read `pubspec.yaml` to get the project package name.
2. List `lib/features/` to detect role directories (customer, admin, business, shared).
3. Read `lib/app/view/app_page.dart` to understand the MultiBlocProvider setup.
4. Read `lib/go_router/router.dart` to understand routing setup.
5. Read `lib/go_router/routes.dart` for route constants pattern.
6. Read `lib/go_router/exports.dart` for the barrel export pattern.

## Step 2: Determine Feature Path

- **With role**: `lib/features/{role}/{role}_{feature}/`
- **Without role**: `lib/features/{feature}/`
- **Class prefix**: `{Role}{Feature}` (e.g., `CustomerOrders`) or just `{Feature}`

## Step 3: Create Feature Files

### 3a: Repository Interface
`domain/repositories/{prefix}_repository.dart`:
```dart
import 'package:{pkg}/utils/helpers/repository_response.dart';

abstract class {Prefix}Repository {
  // Methods will be added as APIs are integrated
}
```

### 3b: Repository Implementation
`data/repositories/{prefix}_repository_impl.dart`:
```dart
import 'package:{pkg}/core/api_service/api_service.dart';
import 'package:{pkg}/core/api_service/app_api_exception.dart';
import 'package:{pkg}/core/endpoints/endpoints.dart';
import 'package:{pkg}/features/{role}/{feature}/domain/repositories/{prefix}_repository.dart';

class {Prefix}RepositoryImpl implements {Prefix}Repository {
  final ApiService _apiService = ApiService();
}
```

### 3c: State
`presentation/cubit/state.dart`:
```dart
import 'package:equatable/equatable.dart';
import 'package:{pkg}/utils/helpers/data_state.dart';

class {Prefix}State extends Equatable {
  const {Prefix}State();

  {Prefix}State copyWith() {
    return const {Prefix}State();
  }

  @override
  List<Object?> get props => [];
}
```

### 3d: Cubit
`presentation/cubit/cubit.dart`:
```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:{pkg}/features/{path}/domain/repositories/{prefix}_repository.dart';
import 'package:{pkg}/features/{path}/presentation/cubit/state.dart';

class {Prefix}Cubit extends Cubit<{Prefix}State> {
  {Prefix}Cubit({required this.repository}) : super(const {Prefix}State());

  final {Prefix}Repository repository;
}
```

### 3e: Screen
`presentation/views/{prefix}_screen.dart`:
```dart
import 'package:{pkg}/exports.dart';
import 'package:{pkg}/features/{path}/presentation/cubit/cubit.dart';
import 'package:{pkg}/features/{path}/presentation/cubit/state.dart';

class {Prefix}Screen extends StatelessWidget {
  const {Prefix}Screen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: mueblyAppBar(context: context, title: '{Feature Title}'),
      body: BlocBuilder<{Prefix}Cubit, {Prefix}State>(
        builder: (context, state) {
          return const Center(
            child: Text('{Feature Title}'),
          );
        },
      ),
    );
  }
}
```

Note: Check the project's actual appBar widget name by reading `utils/widgets/core_widgets/export.dart`.

Always use StatelessWidget for screens — only use StatefulWidget when you have actual mutable state (TextEditingControllers, AnimationControllers). BlocBuilder/BlocListener do NOT require StatefulWidget.

### 3f: Create empty `presentation/widgets/` directory
Create a `.gitkeep` or note that widgets will be extracted here. All complex UI sections must be extracted into separate widget files here — never use private `_build` helper methods in screens. One public widget class per file.

## Step 4: Wire Cubit into MultiBlocProvider

Read `lib/app/view/app_page.dart` and add:

1. Import the cubit:
```dart
import 'package:{pkg}/features/{path}/presentation/cubit/cubit.dart';
```

2. Import the repository impl:
```dart
import 'package:{pkg}/features/{path}/data/repositories/{prefix}_repository_impl.dart';
```

3. Add to the `MultiBlocProvider.providers` list:
```dart
BlocProvider(create: (_) => {Prefix}Cubit(repository: {Prefix}RepositoryImpl())),
```

## Step 5: Wire Route

1. Read `lib/go_router/routes.dart` and add:
```dart
static const String {prefix}Screen = '/{role}-{feature}';
```

2. Read `lib/go_router/router.dart` and add the route entry following the existing pattern (usually `GoRoute` inside a `ShellRoute` or top-level).

3. Read `lib/go_router/exports.dart` and add the screen export:
```dart
export 'package:{pkg}/features/{path}/presentation/views/{prefix}_screen.dart';
```

4. If the project uses `AppRouteNames`, add the route name constant too.

## Step 6: Verify

1. Run `dart analyze lib/` to check for errors.
2. Fix any issues found.
3. Report what was created and how to navigate to the new screen.

## Rules

- Follow the exact file naming and class naming conventions of the existing project.
- Read existing features first to match the exact patterns used.
- ONE cubit, ONE state per feature — always.
- ApiService is a singleton — never inject via constructor.
- Use `const` constructors everywhere possible.
- Match the import style of existing files.
- Business logic belongs in the cubit, not the UI — time calculations, event payloads, refresh orchestration go in cubit methods. Screens only call cubit methods.
