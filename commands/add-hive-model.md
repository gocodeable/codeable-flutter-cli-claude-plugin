---
description: Generate a Hive TypeAdapter model for local persistence — scans for next available TypeId, creates model with @HiveType/@HiveField annotations, registers adapter in bootstrap.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<feature-path> <model-name> e.g. lib/features/profile UserCache"
---

# Add Hive Model

Create a Hive-persisted model with TypeAdapter annotations and register it.

## Arguments

Parse $ARGUMENTS for:
- **Feature path** (required): Where to place the model
- **Model name** (required): e.g., UserCache, SettingsData

Then ask the user for the model fields (name + type pairs).

## Step 1: Find Next Available TypeId

Search the entire project for `@HiveType(typeId:` annotations.
Find the maximum TypeId and use max + 1 for the new model.

## Step 2: Create Hive Model

Create at `<feature-path>/data/models/<model_name_snake>.dart`:

```dart
import 'package:hive_ce/hive_ce.dart';
import 'package:equatable/equatable.dart';

part '<model_name_snake>.g.dart';

@HiveType(typeId: X)
class UserCache extends Equatable {
  const UserCache({
    this.id,
    this.name,
    this.cachedAt,
  });

  @HiveField(0)
  final int? id;

  @HiveField(1)
  final String? name;

  @HiveField(2)
  final DateTime? cachedAt;

  UserCache copyWith({
    int? id,
    String? name,
    DateTime? cachedAt,
  }) {
    return UserCache(
      id: id ?? this.id,
      name: name ?? this.name,
      cachedAt: cachedAt ?? this.cachedAt,
    );
  }

  @override
  List<Object?> get props => [id, name, cachedAt];
}
```

## Step 3: Register Adapter

Find where Hive adapters are registered (usually `bootstrap.dart`).
Add: `Hive.registerAdapter(UserCacheAdapter());`

## Step 4: Check Dependencies

Ensure `hive_ce` is in dependencies and `hive_ce_generator` + `build_runner` are in dev_dependencies.

## Step 5: Instruct User

Tell user to run: `dart run build_runner build --delete-conflicting-outputs`

## Step 6: Verify

Run `dart analyze` (ignore missing .g.dart error until build_runner runs).

## Rules

- TypeId must be UNIQUE across the entire project — always scan first
- HiveField indices start at 0, increment sequentially
- NEVER reuse a TypeId or HiveField index
- Use `hive_ce` (community edition), not the original `hive` package
- Always include the `part` directive for code generation
