---
description: Create a data model from a JSON example — generates Dart class with fromJson, toJson, copyWith, Equatable, and proper null-safe type casting.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<model_name> --json '{...}' [--feature <path>]"
---

# Generate Model from JSON

Create a Dart model class from a JSON example with all codeable_cli conventions.

## Arguments

Parse `$ARGUMENTS` for:
- **Model name** (required): e.g., `OrderModel`, `ProductData`
- **JSON example**: inline or the user will provide it
- **Feature path** (optional): Where to place the model. If omitted, ask if it should go in core or a feature.

## Step 1: Determine Placement

- **Core model** → `lib/core/models/{domain}_models/{name}_model.dart`
- **Feature model** → `lib/features/{role}/{feature}/data/models/{name}_model.dart`

Ask the user if unclear.

## Step 2: Parse JSON

Analyze the JSON structure and generate:
- Field names and types
- Nested object classes
- List types

## Step 3: Generate Model

**Entity model** (has toJson + copyWith):
```dart
import 'package:equatable/equatable.dart';

class OrderModel extends Equatable {
  const OrderModel({
    required this.id,
    required this.name,
    this.description,
  });

  factory OrderModel.fromJson(Map<String, dynamic> json) {
    return OrderModel(
      id: json['id'] as String? ?? '',
      name: json['name'] as String? ?? '',
      description: json['description'] as String?,
    );
  }

  final String id;
  final String name;
  final String? description;

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    if (description != null) 'description': description,
  };

  OrderModel copyWith({
    String? id,
    String? name,
    String? description,
  }) {
    return OrderModel(
      id: id ?? this.id,
      name: name ?? this.name,
      description: description ?? this.description,
    );
  }

  @override
  List<Object?> get props => [id, name, description];
}
```

**Type casting rules:**
- Strings: `json['key'] as String? ?? ''`
- Ints: `(json['key'] as num?)?.toInt() ?? 0`
- Doubles: `(json['key'] as num?)?.toDouble() ?? 0`
- Bools: `json['key'] as bool? ?? false`
- Nullable: `json['key'] as Type?` (no default)
- Lists: `(json['key'] as List<dynamic>?)?.map(...).toList() ?? []`
- Nested: null-check + `Model.fromJson(json['key'] as Map<String, dynamic>)`

## Step 4: Generate Nested Models

If JSON has nested objects, create separate classes (in same file if small, separate file if large).

## Step 5: Verify

Run `dart analyze` on the created file(s).
