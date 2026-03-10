---
description: Add pagination (infinite scroll) to an existing list in a feature — wires up page tracking, scroll listener, pageLoading state, and list append logic.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> <list_field_name>"
---

# Add Pagination

Add infinite scroll pagination to an existing list in a codeable_cli feature.

## Arguments

- `$ARGUMENTS`: Feature path and the state field name holding the list data.

## Step 1: Read Existing Code

1. Read the feature's state, cubit, repository interface, and impl.
2. Identify the list data field and its type.
3. Read the screen/widget that displays the list.

## Step 2: Update State

Add pagination tracking fields:
```dart
const FeatureState({
  this.itemsData = const DataState.initial(),
  this.currentPage = 1,
  this.hasMorePages = true,
});

final DataState<List<ItemModel>> itemsData;
final int currentPage;
final bool hasMorePages;
```

## Step 3: Update Repository

Update the fetch method to accept page/limit:
```dart
// Interface
Future<RepositoryResponse<ItemsData>> fetchItems({int page = 1, int limit = 10});

// Implementation
@override
Future<RepositoryResponse<ItemsData>> fetchItems({int page = 1, int limit = 10}) async {
  try {
    final response = await _apiService.get(
      Endpoints.items,
      queryParams: {'page': page, 'limit': limit},
    );
    // ... parse response
  } on AppApiException catch (e) {
    return RepositoryResponse(isSuccess: false, message: e.message);
  }
}
```

## Step 4: Update Cubit

Add initial fetch and load more methods:
```dart
Future<void> fetchItems() async {
  emit(state.copyWith(
    itemsData: const DataState.loading(),
    currentPage: 1,
    hasMorePages: true,
  ));

  final result = await repository.fetchItems(page: 1);
  if (result.isSuccess && result.data != null) {
    final items = result.data!.items;
    emit(state.copyWith(
      itemsData: DataState.loaded(data: items),
      hasMorePages: items.length >= 10,
    ));
  } else {
    emit(state.copyWith(itemsData: DataState.failure(error: result.message)));
  }
}

Future<void> loadMoreItems() async {
  if (!state.hasMorePages || state.itemsData.isPageLoading) return;

  final nextPage = state.currentPage + 1;
  emit(state.copyWith(itemsData: state.itemsData.toPageLoading()));

  final result = await repository.fetchItems(page: nextPage);
  if (result.isSuccess && result.data != null) {
    final newItems = result.data!.items;
    final allItems = [...(state.itemsData.data ?? []), ...newItems];
    emit(state.copyWith(
      itemsData: DataState.loaded(data: allItems),
      currentPage: nextPage,
      hasMorePages: newItems.length >= 10,
    ));
  } else {
    emit(state.copyWith(
      itemsData: state.itemsData.toLoaded(),
    ));
  }
}
```

## Step 5: Update UI

If the project has `PaginatedListView`, use it:
```dart
PaginatedListView(
  itemCount: items.length,
  hasMore: state.hasMorePages,
  isLoadingMore: state.itemsData.isPageLoading,
  onLoadMore: () => context.read<FeatureCubit>().loadMoreItems(),
  itemBuilder: (context, index) => ItemCard(item: items[index]),
)
```

Otherwise, add a `ScrollController`:
```dart
class _ScreenState extends State<Screen> {
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      context.read<FeatureCubit>().loadMoreItems();
    }
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}
```

## Step 6: Add Pull-to-Refresh

Wrap the list in `RefreshIndicator`:
```dart
RefreshIndicator(
  onRefresh: () => context.read<FeatureCubit>().fetchItems(),
  child: ListView.builder(...),
)
```

## Step 7: Verify

Run `dart analyze lib/` and fix any issues.
