---
name: flutter-bloc-patterns
description: BLoC/Cubit state management patterns for codeable_cli projects — DataState wrapper, Equatable states, copyWith, repository response handling, BlocBuilder/BlocConsumer usage. Use when writing or modifying state management code.
---

# Flutter BLoC/Cubit Patterns (Codeable Style)

You are an expert in the BLoC/Cubit state management patterns used by codeable_cli projects.

## DataState Pattern

All async data is wrapped with `DataState<T>`:

```dart
const DataState.initial()    // Not yet fetched
const DataState.loading()    // Request in flight
DataState.loaded(data: result)  // Success with data
DataState.failure(error: message)  // Error with message
```

Properties:
- `state.data.isInitial` — true when initial
- `state.data.isLoading` — true when loading
- `state.data.isLoaded` — true when loaded
- `state.data.isFailure` — true when failed
- `state.data.data` — the actual data (nullable)
- `state.data.error` — error message (nullable)

## State Pattern

```dart
class FeatureState extends Equatable {
  const FeatureState({
    this.someData = const DataState.initial(),
    this.otherData = const DataState.initial(),
  });

  final DataState<SomeModel> someData;
  final DataState<OtherModel> otherData;

  FeatureState copyWith({
    DataState<SomeModel>? someData,
    DataState<OtherModel>? otherData,
  }) {
    return FeatureState(
      someData: someData ?? this.someData,
      otherData: otherData ?? this.otherData,
    );
  }

  @override
  List<Object?> get props => [someData, otherData];
}
```

Rules:
- Always extend `Equatable`
- ALL async data uses `DataState<T>` — never raw types
- Default all DataState fields to `const DataState.initial()`
- `copyWith` uses null-coalescing (`??`) pattern
- `props` must include ALL fields for proper equality checks
- Non-async state (form fields, flags) can use raw types with defaults

## Cubit Pattern

```dart
class FeatureCubit extends Cubit<FeatureState> {
  FeatureCubit({required this.repository}) : super(const FeatureState());

  final FeatureRepository repository;

  Future<void> fetchData() async {
    emit(state.copyWith(someData: const DataState.loading()));
    final result = await repository.fetchData();
    if (result.isSuccess && result.data != null) {
      emit(state.copyWith(someData: DataState.loaded(data: result.data)));
    } else {
      emit(state.copyWith(someData: DataState.failure(error: result.message)));
    }
  }
}
```

Rules:
- Repository injected via constructor
- Always emit `loading` before async operations
- Check `result.isSuccess && result.data != null` for success
- Use `result.message` for failure error text
- For void operations (DELETE), check `result.isSuccess` only
- **Business logic belongs in the cubit** — time calculations, event payload construction, data grouping/filtering, and refresh orchestration are cubit responsibilities. UI only calls cubit methods and renders state.
- **No success logs** — Don't add `AppLogger.info('X fetched successfully')` after API calls. Only use `AppLogger.error()` for failures. Use `ToastHelper` for user-facing success feedback.
- **Consolidate refresh patterns** — When multiple API calls need to happen after a mutation (create/delete/update), create a single cubit method like `refreshData()` that runs them in parallel with `Future.wait`, rather than calling 3+ methods inline in UI listeners.

## Repository Pattern

Abstract interface:
```dart
abstract class FeatureRepository {
  Future<RepositoryResponse<DataType>> fetchData();
  Future<RepositoryResponse<void>> deleteItem({required String id});
}
```

Implementation:
```dart
class FeatureRepositoryImpl implements FeatureRepository {
  final ApiService _apiService = ApiService(); // Singleton, NOT injected

  @override
  Future<RepositoryResponse<DataType>> fetchData() async {
    try {
      final response = await _apiService.get(Endpoints.featureData);
      final result = ResponseModel.fromApiResponse(
        response,
        DataResponseModel.fromJson,
      );
      if (result.isSuccess) {
        return RepositoryResponse(isSuccess: true, data: result.response?.data);
      }
      return RepositoryResponse(
        isSuccess: false,
        message: result.error ?? 'Failed to fetch data',
      );
    } on AppApiException catch (e) {
      return RepositoryResponse(isSuccess: false, message: e.message);
    }
  }
}
```

Rules:
- ApiService is a singleton — `final ApiService _apiService = ApiService();`
- Only catch `AppApiException` — never bare `Exception`
- Return `RepositoryResponse<T>` always
- Parse with `ResponseModel.fromApiResponse(response, Model.fromJson)`

## UI Patterns

**BlocBuilder** — for rendering based on state:
```dart
BlocBuilder<FeatureCubit, FeatureState>(
  builder: (context, state) {
    if (state.someData.isLoading) return const LoadingWidget();
    if (state.someData.isFailure) {
      return RetryWidget(
        message: state.someData.error,
        onRetry: () => context.read<FeatureCubit>().fetchData(),
      );
    }
    if (!state.someData.isLoaded) return const SizedBox.shrink();
    final data = state.someData.data!;
    return DataWidget(data: data);
  },
)
```

**BlocConsumer** — when you need both builder and listener (e.g., show snackbar on success):
```dart
BlocConsumer<FeatureCubit, FeatureState>(
  listenWhen: (prev, curr) => prev.submitData != curr.submitData,
  listener: (context, state) {
    if (state.submitData.isLoaded) {
      ScaffoldMessenger.of(context).showSnackBar(...);
    }
  },
  builder: (context, state) { ... },
)
```

**Reading cubit**:
- `context.read<Cubit>()` — one-time reads (in callbacks, initState)
- `context.watch<Cubit>()` — reactive rebuilds (in build methods)

**StatelessWidget by default** — Only use StatefulWidget when you have actual mutable state (TextEditingControllers, ScrollControllers, AnimationControllers, etc.). BlocBuilder/BlocListener do NOT require StatefulWidget.

**Button loading state**:
```dart
MueblyButton(
  text: context.l10n.submit,
  isLoading: state.submitData.isLoading,
  onPressed: () => cubit.submit(),
)
```
Don't redundantly disable buttons — `MueblyButton` (and `RMBButton`) already handles disabled state when `isLoading: true` is set (internally does `(isLoading || disabled) ? null : onPressed`). Don't also set `disabled: state.submitData.isLoading` or `onPressed: isLoading ? null : handler` — it's redundant.

## Model Patterns

**Response model** (extends BaseApiResponse):
```dart
class DataResponseModel extends BaseApiResponse<DataClass> {
  DataResponseModel({required super.statusCode, super.error, super.data});

  factory DataResponseModel.fromJson(Map<String, dynamic> json) {
    final base = BaseApiResponse<DataClass>.fromJson(json, DataClass.fromJson);
    return DataResponseModel(
      statusCode: base.statusCode, error: base.error, data: base.data,
    );
  }
}
```

**Entity model** (with toJson + copyWith):
```dart
class ItemModel extends Equatable {
  const ItemModel({required this.id, required this.name});

  factory ItemModel.fromJson(Map<String, dynamic> json) {
    return ItemModel(
      id: json['id'] as String? ?? '',
      name: json['name'] as String? ?? '',
    );
  }

  final String id;
  final String name;

  Map<String, dynamic> toJson() => {'id': id, 'name': name};

  ItemModel copyWith({String? id, String? name}) {
    return ItemModel(id: id ?? this.id, name: name ?? this.name);
  }

  @override
  List<Object?> get props => [id, name];
}
```

**Type casting rules:**
- Strings: `json['key'] as String? ?? ''`
- Ints: `(json['key'] as num?)?.toInt() ?? 0`
- Doubles: `(json['key'] as num?)?.toDouble() ?? 0`
- Bools: `json['key'] as bool? ?? false`
- Lists: `(json['key'] as List<dynamic>?)?.map((e) => Model.fromJson(e as Map<String, dynamic>)).toList() ?? []`
- Nested: null-check then `Model.fromJson(json['key'] as Map<String, dynamic>)`
