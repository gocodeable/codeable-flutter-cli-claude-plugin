---
description: Wire a new or existing repository and cubit into the dependency injection and MultiBlocProvider setup.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "<feature_path>"
---

# Wire Dependency Injection

Wire a feature's repository and cubit into the project's DI and MultiBlocProvider.

## Arguments

- `$ARGUMENTS`: Path to the feature (e.g., `lib/features/customer/customer_orders`).

## Steps

1. Read `pubspec.yaml` for package name.
2. Read the feature's cubit and repository files to get class names.
3. Read `lib/app/view/app_page.dart` to find the MultiBlocProvider.
4. Check if the cubit is already registered. If so, report and stop.
5. Add the cubit registration:

```dart
BlocProvider(create: (_) => {Prefix}Cubit(repository: {Prefix}RepositoryImpl())),
```

6. Add required imports for the cubit and repository impl.
7. If the project uses GetIt for repository DI, also register there.
8. Run `dart analyze lib/` to verify.
