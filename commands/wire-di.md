---
description: Wire a new or existing repository, service, or cubit into the dependency injection and MultiBlocProvider setup.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<feature_path>"
---

# Wire Dependency Injection

Wire a feature's repository and cubit (or standalone service) into the project's DI and MultiBlocProvider.

## Arguments

- `$ARGUMENTS`: Path to the feature (e.g., `lib/features/customer/customer_orders`) or service (e.g., `lib/core/services/analytics_service`).

## Step 1: Determine DI Path

Ask the user (or infer from the path) which type:

### Path A: Feature Repository + Cubit
For features under `lib/features/` -- wire both repository and cubit.

### Path B: Standalone Service
For services under `lib/core/services/` -- wire only the service into GetIt.

## Step 2: Read Existing Code

1. Read `pubspec.yaml` for package name.
2. Read the feature's cubit and repository files (or service files) to get class names.
3. Read `lib/app/view/app_page.dart` to find the MultiBlocProvider.
4. Read the DI setup file (usually `lib/app/injector.dart` or similar) to find the GetIt configuration.

## Step 3: Register in GetIt (Repository or Service)

### For Feature Repository:

Register the repository implementation against its abstract interface:

```dart
injector.registerLazySingleton<FeatureRepository>(
  () => FeatureRepositoryImpl(
    apiService: injector<ApiService>(),
  ),
);
```

### For Standalone Service:

```dart
injector.registerLazySingleton<AnalyticsService>(
  () => AnalyticsServiceImpl(),
);
```

**Pattern:** `injector.registerLazySingleton<AbstractType>(() => ConcreteType())`

Dependencies are resolved via: `injector<DependencyType>()`

## Step 4: Register Cubit in MultiBlocProvider (Feature only)

Check if the cubit is already registered. If so, report and stop.

Add the cubit registration in `app_page.dart`:

```dart
BlocProvider(
  create: (_) => FeatureCubit(
    repository: injector<FeatureRepository>(),
  ),
),
```

Note: The cubit receives its repository from the injector, NOT created inline.

## Step 5: Add Imports

Add required imports for:
- The cubit class
- The repository interface (for the injector type)
- The repository implementation (for the injector factory)
- Any service dependencies

## Step 6: Verify

Run `dart analyze lib/` to verify no import errors or missing dependencies.

## Rules

- Always register the **abstract type** in GetIt, instantiating the **concrete type**: `injector.registerLazySingleton<Abstract>(() => Concrete())`.
- Resolve dependencies via `injector<DependencyType>()`, never by constructing them directly in the registration.
- Cubits go in `MultiBlocProvider` in `app_page.dart`, repositories/services go in the `injector` setup.
- Check for duplicates before adding -- never double-register.
- If the project uses a different DI approach (e.g., Riverpod, Provider), adapt accordingly.
