---
description: Add pagination (infinite scroll) to an existing list in a feature — uses PaginationModel in cubit state, pageLoading for appending, and PaginatedListView from core widgets.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<feature_path> <list_field_name>"
---

# Add Pagination

Add infinite scroll pagination to an existing list in a codeable_cli feature using a **PaginationModel** approach.

## Arguments

- `$ARGUMENTS`: Feature path and the state field name holding the list data.

## Step 1: Read Existing Code

1. Read the feature's state, cubit, repository interface, and impl.
2. Identify the list data field and its type (e.g., `DataState<List<ItemModel>>`).
3. Read the screen/widget that displays the list.
4. Check if a `PaginationModel` already exists in the project (search `lib/core/models/`).

## Step 2: Create PaginationModel (if not exists)

If the project does not already have a `PaginationModel`, create it at `lib/core/models/pagination_model.dart`:

```dart
import 'package:equatable/equatable.dart';

class PaginationModel<T> extends Equatable {
  const PaginationModel({
    this.items = const [],
    this.currentPage = 1,
    this.totalPages = 1,
    this.perPage = 10,
  });

  final List<T> items;
  final int currentPage;
  final int totalPages;
  final int perPage;

  bool get hasMore => currentPage < totalPages;

  PaginationModel<T> copyWith({
    List<T>? items,
    int? currentPage,
    int? totalPages,
    int? perPage,
  }) {
    return PaginationModel<T>(
      items: items ?? this.items,
      currentPage: currentPage ?? this.currentPage,
      totalPages: totalPages ?? this.totalPages,
      perPage: perPage ?? this.perPage,
    );
  }

  @override
  List<Object?> get props => [items, currentPage, totalPages, perPage];
}
```

## Step 3: Update State

Replace the separate list field with a `PaginationModel` wrapped in `DataState`:

```dart
const FeatureState({
  this.itemsData = const DataState.initial(),
});

final DataState<PaginationModel<ItemModel>> itemsData;
```

Do NOT use separate `currentPage` / `hasMorePages` fields. Everything lives inside `PaginationModel`.

## Step 4: Update Repository

Update the fetch method to accept page/limit and return pagination metadata:
```dart
// Interface
Future<RepositoryResponse<PaginationModel<ItemModel>>> fetchItems({int page = 1, int limit = 10});

// Implementation
@override
Future<RepositoryResponse<PaginationModel<ItemModel>>> fetchItems({int page = 1, int limit = 10}) async {
  try {
    final response = await _apiService.get(
      Endpoints.items,
      queryParams: {'page': page, 'limit': limit},
    );
    final data = response.data as Map<String, dynamic>;
    final items = (data['data'] as List<dynamic>?)
            ?.map((e) => ItemModel.fromJson(e as Map<String, dynamic>))
            .toList() ??
        [];
    final pagination = PaginationModel<ItemModel>(
      items: items,
      currentPage: (data['current_page'] as num?)?.toInt() ?? page,
      totalPages: (data['last_page'] as num?)?.toInt() ?? 1,
      perPage: (data['per_page'] as num?)?.toInt() ?? limit,
    );
    return RepositoryResponse(isSuccess: true, data: pagination);
  } on AppApiException catch (e) {
    return RepositoryResponse(isSuccess: false, message: e.message);
  }
}
```

## Step 5: Update Cubit

Add three methods: `fetchItems()`, `loadNextPage()`, and `refresh()`:

```dart
Future<void> fetchItems() async {
  emit(state.copyWith(
    itemsData: const DataState.loading(),
  ));

  final result = await repository.fetchItems(page: 1);
  if (result.isSuccess && result.data != null) {
    emit(state.copyWith(
      itemsData: DataState.loaded(data: result.data!),
    ));
  } else {
    emit(state.copyWith(
      itemsData: DataState.failure(error: result.message),
    ));
  }
}

Future<void> loadNextPage() async {
  final currentData = state.itemsData.data;
  if (currentData == null || !currentData.hasMore || state.itemsData.isPageLoading) return;

  emit(state.copyWith(
    itemsData: state.itemsData.toPageLoading(),
  ));

  final nextPage = currentData.currentPage + 1;
  final result = await repository.fetchItems(page: nextPage);

  if (result.isSuccess && result.data != null) {
    final newPageData = result.data!;
    final allItems = [...currentData.items, ...newPageData.items];
    emit(state.copyWith(
      itemsData: DataState.loaded(
        data: currentData.copyWith(
          items: allItems,
          currentPage: newPageData.currentPage,
          totalPages: newPageData.totalPages,
        ),
      ),
    ));
  } else {
    // Revert to loaded state on failure (keep existing data)
    emit(state.copyWith(
      itemsData: DataState.loaded(data: currentData),
    ));
  }
}

Future<void> refresh() async {
  await fetchItems();
}
```

## Step 6: Update UI

Wire to `PaginatedListView` from core widgets:
```dart
BlocBuilder<FeatureCubit, FeatureState>(
  builder: (context, state) {
    final itemsData = state.itemsData;
    if (itemsData.isLoading) {
      return const Center(child: CircularProgressIndicator());
    }
    if (itemsData.isFailure) {
      return Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(itemsData.errorMessage ?? context.l10n.somethingWentWrong),
            const SizedBox(height: 8),
            CustomButton(
              text: context.l10n.retry,
              onPressed: () => context.read<FeatureCubit>().fetchItems(),
            ),
          ],
        ),
      );
    }
    final paginationModel = itemsData.data;
    if (paginationModel == null || paginationModel.items.isEmpty) {
      return Center(child: Text(context.l10n.noItemsFound));
    }
    return RefreshIndicator(
      onRefresh: () => context.read<FeatureCubit>().refresh(),
      child: PaginatedListView(
        itemCount: paginationModel.items.length,
        hasMore: paginationModel.hasMore,
        isLoadingMore: itemsData.isPageLoading,
        onLoadMore: () => context.read<FeatureCubit>().loadNextPage(),
        itemBuilder: (context, index) => ItemCard(item: paginationModel.items[index]),
      ),
    );
  },
)
```

## Step 7: Verify

Run `dart analyze lib/` and fix any issues.

## Rules

- Store `DataState<PaginationModel<ItemModel>>` in state, NOT separate `currentPage`/`hasMorePages` fields.
- `loadNextPage` uses `DataState.pageLoading` and APPENDS items to existing list.
- `refresh` resets to page 1 (calls `fetchItems()`).
- Guard `loadNextPage` against: no data, no more pages, already page-loading.
- Use `PaginatedListView` from core widgets when available.
- Parse pagination metadata (current_page, last_page, per_page) from API response.
