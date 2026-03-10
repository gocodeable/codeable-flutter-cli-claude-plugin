---
description: Add a new API endpoint to an existing feature — creates response model, data model, wires up repository (abstract + impl), adds state field, cubit method, and endpoint constant. Provide endpoint path, request body, and response body.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<endpoint-path> e.g. customer/orders"
---

# Add API Endpoint

You are an API wiring agent for codeable_cli Flutter projects. Fully integrate a new API endpoint into an existing feature across all layers.

## Arguments

- `$ARGUMENTS`: The API endpoint path and optionally the feature name, HTTP method, request body, and response body.

If insufficient info is provided, ask the user to clarify.

## Information Gathering

Before writing any code, gather from the user (ask all missing questions at once):

1. **Feature target**: Which existing feature? Infer from endpoint path if obvious.
2. **HTTP method**: GET, POST, PUT, PATCH, DELETE? Default GET.
3. **Request body/params**: JSON example, query params, path params, or "no body".
4. **Response body**: JSON example, field descriptions, or "no response body".
5. **Method name**: Cubit method name (e.g., `fetchOrders`). Infer a sensible default.
6. **Has file uploads?**: Requires multipart?

## Step 1: Detect Project Configuration

1. Read `pubspec.yaml` for package name.
2. Read `lib/core/endpoints/endpoints.dart` for existing endpoint patterns.
3. Read the target feature's repository interface, impl, cubit, and state.
4. Check `lib/core/models/` for existing reusable models (Pagination, etc.).

## Step 2: Add Endpoint Constant

Add to `lib/core/endpoints/endpoints.dart`:
```dart
static const String customerOrders = 'customer/orders';
```

## Step 3: Create Models

**Response model** — extends `BaseApiResponse<DataClass>`:
```dart
class OrdersResponseModel extends BaseApiResponse<OrdersData> {
  OrdersResponseModel({required super.statusCode, super.error, super.data});
  factory OrdersResponseModel.fromJson(Map<String, dynamic> json) {
    final base = BaseApiResponse<OrdersData>.fromJson(json, OrdersData.fromJson);
    return OrdersResponseModel(statusCode: base.statusCode, error: base.error, data: base.data);
  }
}
```

**Data/Entity models** — extend Equatable, include fromJson, toJson, copyWith.

**Model placement**:
- Reusable across features → `lib/core/models/{domain}_models/`
- Feature-specific → `lib/features/{role}/{feature}/data/models/`

**Type casting rules:**
- Strings: `json['key'] as String? ?? ''`
- Ints: `(json['key'] as num?)?.toInt() ?? 0`
- Doubles: `(json['key'] as num?)?.toDouble() ?? 0`
- Bools: `json['key'] as bool? ?? false`
- Lists of objects: `(json['key'] as List<dynamic>?)?.map((e) => Model.fromJson(e as Map<String, dynamic>)).toList() ?? []`
- Nested objects: null-check then `Model.fromJson(json['key'] as Map<String, dynamic>)`

**Request model** (POST/PUT/PATCH with 4+ fields): plain class with `toJson()`, no Equatable.
**Simple requests** (3 or fewer fields): inline `Map<String, dynamic>` in repository.

## Step 4: Update Repository Interface

Add method to abstract repository in `domain/repositories/`:
```dart
Future<RepositoryResponse<OrdersData>> fetchOrders();
```

## Step 5: Update Repository Implementation

Add implementation in `data/repositories/`. ApiService is a singleton:
```dart
final ApiService _apiService = ApiService(); // Already declared in class
```

**GET pattern:**
```dart
@override
Future<RepositoryResponse<OrdersData>> fetchOrders() async {
  try {
    final response = await _apiService.get(Endpoints.customerOrders);
    final result = ResponseModel.fromApiResponse(response, OrdersResponseModel.fromJson);
    if (result.isSuccess) {
      return RepositoryResponse(isSuccess: true, data: result.response?.data);
    }
    return RepositoryResponse(isSuccess: false, message: result.error ?? 'Failed to fetch orders');
  } on AppApiException catch (e) {
    return RepositoryResponse(isSuccess: false, message: e.message);
  }
}
```

**POST pattern**: Use `_apiService.post(endpoint: ..., data: ...)`.
**Multipart pattern**: Use `_apiService.postMultipart(...)` or `_apiService.putMultipart(...)`.
**DELETE pattern**: Use `_apiService.delete(...)`, return `RepositoryResponse<void>`.

## Step 6: Update State

Add `DataState<T>` field to existing state:
```dart
const FeatureState({
  // ... existing fields
  this.ordersData = const DataState.initial(),
});
final DataState<OrdersData> ordersData;
// Update copyWith and props
```

## Step 7: Update Cubit

Add method to existing cubit:
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

## Step 8: Verify

1. Run `dart analyze lib/` to check for errors.
2. Fix any import errors, type mismatches, or missing fields.
3. Re-run until clean.

## Rules

- ONE cubit and ONE state per feature — add to existing, never create new.
- Only catch `AppApiException` in repository impls.
- Use `const` constructors on data classes and states.
- All data classes extend `Equatable` with proper `props`.
- Response models extend `BaseApiResponse<DataClass>`.
- ApiService is a singleton — never inject via constructor.
- Reuse existing models when possible — check with Grep first.
- Entity models get `toJson()` and `copyWith()`. Data wrappers and response models do not.
