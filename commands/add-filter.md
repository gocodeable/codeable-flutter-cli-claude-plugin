---
description: Add filter/sort UI to an existing list — filter chips, sort dropdown, filter bottom sheet, and state management for active filters.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> --filters 'category,price_range,status'"
---

# Add Filters to List

Add filtering and sorting to an existing list view in a feature.

## Arguments

- `$ARGUMENTS`: Feature path and filter field names.

If filters not specified, ask the user what fields to filter by.

## Step 1: Read Existing Code

Read the feature's cubit, state, and list screen.

## Step 2: Update State

Add filter fields:
```dart
const FeatureState({
  this.itemsData = const DataState.initial(),
  this.selectedCategory = '',
  this.selectedStatus = '',
  this.sortBy = 'newest',
  this.filtersData = const DataState.initial(), // For API-fetched filter options
});

final String selectedCategory;
final String selectedStatus;
final String sortBy;
final DataState<FiltersModel> filtersData;
```

## Step 3: Update Cubit

```dart
void setCategory(String category) {
  emit(state.copyWith(selectedCategory: category));
  fetchItems(); // Re-fetch with filters
}

void setSortBy(String sort) {
  emit(state.copyWith(sortBy: sort));
  fetchItems();
}

void clearFilters() {
  emit(state.copyWith(
    selectedCategory: '',
    selectedStatus: '',
    sortBy: 'newest',
  ));
  fetchItems();
}

int get activeFilterCount {
  int count = 0;
  if (state.selectedCategory.isNotEmpty) count++;
  if (state.selectedStatus.isNotEmpty) count++;
  if (state.sortBy != 'newest') count++;
  return count;
}
```

## Step 4: Update Repository

Pass filters as query params:
```dart
Future<RepositoryResponse<ItemsData>> fetchItems({
  String? category,
  String? status,
  String sortBy = 'newest',
  int page = 1,
}) async {
  final queryParams = <String, dynamic>{
    'page': page,
    'sort': sortBy,
    if (category != null && category.isNotEmpty) 'category': category,
    if (status != null && status.isNotEmpty) 'status': status,
  };
  final response = await _apiService.get(Endpoints.items, queryParams: queryParams);
  // ...
}
```

## Step 5: Create Filter UI

### Filter Chips (horizontal row):
```dart
SingleChildScrollView(
  scrollDirection: Axis.horizontal,
  padding: const EdgeInsets.symmetric(horizontal: 16),
  child: Row(
    children: [
      FilterChip(
        label: Text('Category'),
        selected: state.selectedCategory.isNotEmpty,
        onSelected: (_) => _showCategoryPicker(context),
      ),
      const SizedBox(width: 8),
      FilterChip(
        label: Text('Sort: ${state.sortBy}'),
        selected: state.sortBy != 'newest',
        onSelected: (_) => _showSortOptions(context),
      ),
      if (cubit.activeFilterCount > 0) ...[
        const SizedBox(width: 8),
        ActionChip(
          label: const Text('Clear All'),
          onPressed: () => cubit.clearFilters(),
        ),
      ],
    ],
  ),
)
```

### Filter Bottom Sheet:
Create `presentation/widgets/{prefix}_filter_sheet.dart` with filter options and Apply/Clear buttons.

## Step 6: Verify

Run `dart analyze lib/` and fix any issues.
