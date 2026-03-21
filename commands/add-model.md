---
description: Create a data model from a JSON example or field descriptions — generates Dart class with fromJson, toJson, copyWith, Equatable, nested model handling, and proper null-safe type casting.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<model_name> --json '{...}' [--feature <path>]"
---

# Generate Model from JSON or Field Descriptions

Create a Dart model class from a JSON example or field descriptions with all codeable_cli conventions.

## Arguments

Parse `$ARGUMENTS` for:
- **Model name** (required): e.g., `OrderModel`, `ProductData`
- **JSON example**: inline or the user will provide it
- **Feature path** (optional): Where to place the model. If omitted, ask if it should go in core or a feature.

If no JSON is provided, ask the user:
- Do you have a **JSON example**, or would you like to describe the **fields manually** (name + type pairs)?

If the user chooses field descriptions, ask for each field:
- Field name (e.g., `id`, `userName`, `createdAt`)
- Field type (e.g., `String`, `int`, `DateTime`, `List<TagModel>`, `AddressModel?`)

## Step 1: Determine Placement

- **Core model** -> `lib/core/models/{domain}_models/{name}_model.dart`
- **Feature model** -> `lib/features/{role}/{feature}/data/models/{name}_model.dart`

Ask the user if unclear.

## Step 2: Parse JSON or Fields

If JSON provided, analyze the JSON structure and generate:
- Field names and types
- Nested object classes
- List types

If fields provided manually, use the name + type pairs directly.

## Step 3: Generate Model

**Entity model** (has toJson + copyWith):
```dart
import 'package:equatable/equatable.dart';

class OrderModel extends Equatable {
  const OrderModel({
    required this.id,
    required this.name,
    this.description,
    this.tags,
    this.address,
    this.createdAt,
  });

  factory OrderModel.fromJson(Map<String, dynamic> json) {
    return OrderModel(
      id: json['id'] as String? ?? '',
      name: json['name'] as String? ?? '',
      description: json['description'] as String?,
      tags: (json['tags'] as List<dynamic>?)
              ?.whereType<Map<String, dynamic>>()
              .map((e) {
                try { return TagModel.fromJson(e); } catch (_) { return null; }
              })
              .whereType<TagModel>()
              .toList() ??
          [],
      address: json['address'] != null
          ? AddressModel.fromJson(json['address'] as Map<String, dynamic>)
          : null,
      createdAt: json['created_at'] != null
          ? DateTime.tryParse(json['created_at'] as String? ?? '')
          : null,
    );
  }

  final String id;
  final String name;
  final String? description;
  final List<TagModel>? tags;
  final AddressModel? address;
  final DateTime? createdAt;

  Map<String, dynamic> toJson() => {
        'id': id,
        'name': name,
        if (description != null) 'description': description,
        if (tags != null) 'tags': tags!.map((e) => e.toJson()).toList(),
        if (address != null) 'address': address!.toJson(),
        if (createdAt != null) 'created_at': createdAt!.toIso8601String(),
      };

  OrderModel copyWith({
    String? id,
    String? name,
    String? description,
    List<TagModel>? tags,
    AddressModel? address,
    DateTime? createdAt,
  }) {
    return OrderModel(
      id: id ?? this.id,
      name: name ?? this.name,
      description: description ?? this.description,
      tags: tags ?? this.tags,
      address: address ?? this.address,
      createdAt: createdAt ?? this.createdAt,
    );
  }

  @override
  List<Object?> get props => [id, name, description, tags, address, createdAt];
}
```

**Type casting rules:**
- Strings: `json['key'] as String? ?? ''`
- Ints: `(json['key'] as num?)?.toInt() ?? 0`
- Doubles: `(json['key'] as num?)?.toDouble() ?? 0`
- Bools: `json['key'] as bool? ?? false`
- Nullable: `json['key'] as Type?` (no default)
- Lists: `(json['key'] as List<dynamic>?)?.map(...).toList() ?? []`
- `List<T>` of models (safe parsing): `?.whereType<Map<String, dynamic>>().map((e) { try { return T.fromJson(e); } catch (_) { return null; } }).whereType<T>().toList() ?? []`
- `List<String>`: `?.map((e) => e as String).toList() ?? []`
- Nested objects: null-check + `Model.fromJson(json['key'] as Map<String, dynamic>)`
- DateTime: `json['key'] != null ? DateTime.tryParse(json['key'] as String? ?? '') : null`

## Step 4: Generate Nested Models

If JSON has nested objects, create separate classes:
- **Small nested models** (2-3 fields): place in the same file below the parent model.
- **Larger nested models** (4+ fields): create a separate file and import it.
- For `List<T>` where T is a nested object, always create a separate model class for T.

## Step 5: Offer Repository Integration

After creating the model, ask the user:
> Would you like me to wire this model into an existing repository method, or create a new repository fetch method for it?

If yes, read the feature's repository interface and implementation and add the appropriate method.

## Step 6: Verify

Run `dart analyze` on the created file(s).

## Rules

- All fields nullable-safe with defensive casting.
- Use `snake_case` JSON keys mapped to `camelCase` Dart fields.
- Always include `fromJson`, `toJson`, `copyWith`, and `Equatable`.
- DateTime fields use `DateTime.tryParse` (never `DateTime.parse`).
- Nested models get their own `fromJson`/`toJson`.
- `List<T>` fields default to `[]`, not `null`, unless explicitly nullable.
