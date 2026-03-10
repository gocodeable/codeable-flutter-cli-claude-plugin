---
description: Add a new state field and cubit method to an existing feature's cubit — handles DataState typing, copyWith, props, and the async method pattern.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> <field_name> <DataType>"
---

# Add Cubit State Field & Method

Add a new `DataState<T>` field to an existing feature's state and a corresponding async method to the cubit.

## Arguments

Parse `$ARGUMENTS` for:
- **Feature path**: e.g., `lib/features/customer/customer_orders`
- **Field name**: e.g., `ordersData`, `deleteOrderStatus`
- **Data type**: e.g., `OrdersData`, `void`

If missing, ask the user.

## Step 1: Read Existing Files

1. Read the feature's `presentation/cubit/state.dart`.
2. Read the feature's `presentation/cubit/cubit.dart`.
3. Read the feature's repository interface and impl.

## Step 2: Update State

Add to `state.dart`:

1. Add field to constructor with default:
```dart
this.ordersData = const DataState.initial(),
```

2. Add field declaration:
```dart
final DataState<OrdersData> ordersData;
```

3. Add to `copyWith` parameter and body:
```dart
DataState<OrdersData>? ordersData,
// ...
ordersData: ordersData ?? this.ordersData,
```

4. Add to `props`:
```dart
@override
List<Object?> get props => [...existing, ordersData];
```

5. Add import for the data type if needed.

## Step 3: Add Repository Method (if API-backed)

If the state field is backed by an API call:

1. Add method to abstract repository.
2. Add implementation to repository impl.
3. Add endpoint to `endpoints.dart` if needed.

## Step 4: Add Cubit Method

Add to `cubit.dart`:
```dart
Future<void> fetchOrders() async {
  emit(state.copyWith(ordersData: const DataState.loading()));
  final result = await repository.fetchOrders();
  if (result.isSuccess && result.data != null) {
    emit(state.copyWith(ordersData: DataState.loaded(data: result.data)));
  } else {
    emit(state.copyWith(ordersData: DataState.failure(error: result.message)));
  }
}
```

For `void` operations:
```dart
Future<void> deleteOrder({required String orderId}) async {
  emit(state.copyWith(deleteOrderStatus: const DataState.loading()));
  final result = await repository.deleteOrder(orderId: orderId);
  if (result.isSuccess) {
    emit(state.copyWith(deleteOrderStatus: const DataState.loaded(data: null)));
  } else {
    emit(state.copyWith(deleteOrderStatus: DataState.failure(error: result.message)));
  }
}
```

## Step 5: Verify

Run `dart analyze lib/` and fix any issues.

## Rules

- NEVER create new cubit/state files — add to existing.
- All async data uses `DataState<T>`.
- Default to `const DataState.initial()`.
- Include ALL fields in `props`.
