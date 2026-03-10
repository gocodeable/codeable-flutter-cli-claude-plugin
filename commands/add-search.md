---
description: Add search/filter functionality to an existing list — wires up search field, debounced query, filtered results in state, and API or local filtering.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> [--api|--local]"
---

# Add Search to List

Add search/filter functionality to an existing list in a feature.

## Arguments

- `$ARGUMENTS`: Feature path and mode (`--api` for server-side search, `--local` for client-side filtering). Default: ask user.

## API Search Mode

### State
```dart
const FeatureState({
  this.itemsData = const DataState.initial(),
  this.searchQuery = '',
  this.searchResults = const DataState.initial(),
});

final DataState<List<ItemModel>> itemsData;
final String searchQuery;
final DataState<List<ItemModel>> searchResults;
```

### Cubit
```dart
Timer? _searchDebounce;

void onSearchChanged(String query) {
  _searchDebounce?.cancel();
  emit(state.copyWith(searchQuery: query));

  if (query.trim().isEmpty) {
    emit(state.copyWith(searchResults: const DataState.initial()));
    return;
  }

  _searchDebounce = Timer(const Duration(milliseconds: 500), () {
    searchItems(query.trim());
  });
}

Future<void> searchItems(String query) async {
  emit(state.copyWith(searchResults: const DataState.loading()));
  final result = await repository.searchItems(query: query);
  if (result.isSuccess && result.data != null) {
    emit(state.copyWith(searchResults: DataState.loaded(data: result.data)));
  } else {
    emit(state.copyWith(searchResults: DataState.failure(error: result.message)));
  }
}

void clearSearch() {
  _searchDebounce?.cancel();
  emit(state.copyWith(
    searchQuery: '',
    searchResults: const DataState.initial(),
  ));
}

@override
Future<void> close() {
  _searchDebounce?.cancel();
  return super.close();
}
```

### Repository
```dart
Future<RepositoryResponse<List<ItemModel>>> searchItems({required String query});
```

## Local Search Mode

### State
```dart
const FeatureState({
  this.itemsData = const DataState.initial(),
  this.searchQuery = '',
});
```

### Cubit
```dart
void onSearchChanged(String query) {
  emit(state.copyWith(searchQuery: query));
}

void clearSearch() {
  emit(state.copyWith(searchQuery: ''));
}

// Computed getter for filtered results
List<ItemModel> get filteredItems {
  final items = state.itemsData.data ?? [];
  if (state.searchQuery.isEmpty) return items;
  final query = state.searchQuery.toLowerCase();
  return items.where((item) =>
    item.name.toLowerCase().contains(query) ||
    item.description.toLowerCase().contains(query)
  ).toList();
}
```

## UI

```dart
Column(
  children: [
    MueblySearchField(
      hintText: 'Search...',
      onChanged: (query) => context.read<FeatureCubit>().onSearchChanged(query),
      onClear: () => context.read<FeatureCubit>().clearSearch(),
    ),
    Expanded(
      child: ListView.builder(
        itemCount: filteredItems.length,
        itemBuilder: (context, index) => ItemCard(item: filteredItems[index]),
      ),
    ),
  ],
)
```

## Rules

- Use 500ms debounce for API search to avoid excessive requests.
- Cancel debounce timer in `close()`.
- Clear search results when query is empty.
- For local search, use computed getter instead of separate state field.
