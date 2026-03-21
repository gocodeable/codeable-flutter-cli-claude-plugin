---
description: Add local caching layer to a feature using Hive — caches API responses locally for offline support and faster load times with cache-then-network strategy.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> <data_key>"
---

# Add Local Cache

Add Hive-based caching to a feature for offline support and faster loads.

## Arguments

- `$ARGUMENTS`: Feature path and cache key name.

## Strategy: Cache-Then-Network

1. Load cached data immediately (instant UI)
2. Fetch fresh data from API in background
3. Update cache + UI when fresh data arrives

## Step 1: Add Cache Methods to AppPreferences

Or create a feature-specific cache class extending `BaseStorage`:

```dart
// Option A: Add to AppPreferences
Future<void> cacheItems(List<ItemModel> items) async {
  final jsonList = items.map((e) => e.toJson()).toList();
  await store('cached_items', jsonEncode(jsonList));
}

List<ItemModel>? getCachedItems() {
  final data = retrieve<String>('cached_items');
  if (data == null) return null;
  final jsonList = jsonDecode(data) as List<dynamic>;
  return jsonList
      .whereType<Map<String, dynamic>>()
      .map((e) {
        try { return ItemModel.fromJson(e); } catch (_) { return null; }
      })
      .whereType<ItemModel>()
      .toList();
}

Future<void> clearItemsCache() => remove('cached_items');
```

```dart
// Option B: Feature-specific cache
class ItemsCache extends BaseStorage {
  Future<void> init() => super.init('items-cache');
  // ... cache methods
}
```

## Step 2: Update Repository

```dart
class FeatureRepositoryImpl implements FeatureRepository {
  final ApiService _apiService = ApiService();
  final AppPreferences _prefs = Injector.resolve<AppPreferences>();

  @override
  Future<RepositoryResponse<List<ItemModel>>> fetchItems({
    bool forceRefresh = false,
  }) async {
    // Return cached data first (if available and not forcing refresh)
    if (!forceRefresh) {
      final cached = _prefs.getCachedItems();
      if (cached != null && cached.isNotEmpty) {
        return RepositoryResponse(isSuccess: true, data: cached);
      }
    }

    try {
      final response = await _apiService.get(Endpoints.items);
      final result = ResponseModel.fromApiResponse(
        response, ItemsResponseModel.fromJson,
      );
      if (result.isSuccess && result.response?.data != null) {
        final items = result.response!.data!.items;
        // Cache the fresh data
        await _prefs.cacheItems(items);
        return RepositoryResponse(isSuccess: true, data: items);
      }
      return RepositoryResponse(isSuccess: false, message: result.error);
    } on AppApiException catch (e) {
      // On network error, return cached data as fallback
      final cached = _prefs.getCachedItems();
      if (cached != null && cached.isNotEmpty) {
        return RepositoryResponse(isSuccess: true, data: cached);
      }
      return RepositoryResponse(isSuccess: false, message: e.message);
    }
  }
}
```

## Step 3: Update Cubit

```dart
Future<void> fetchItems({bool forceRefresh = false}) async {
  emit(state.copyWith(itemsData: const DataState.loading()));

  // Load cached first
  final cachedResult = await repository.fetchItems();
  if (cachedResult.isSuccess && cachedResult.data != null) {
    emit(state.copyWith(itemsData: DataState.loaded(data: cachedResult.data)));
  }

  // Then refresh from network
  if (!forceRefresh && cachedResult.isSuccess) {
    final freshResult = await repository.fetchItems(forceRefresh: true);
    if (freshResult.isSuccess && freshResult.data != null) {
      emit(state.copyWith(itemsData: DataState.loaded(data: freshResult.data)));
    }
  }
}
```

## Step 4: Cache Invalidation

```dart
// Clear on logout
prefs.clearItemsCache();

// Clear on data mutation (create/update/delete)
Future<void> deleteItem({required String id}) async {
  // ... delete API call
  await _prefs.clearItemsCache(); // Invalidate cache
  fetchItems(forceRefresh: true); // Refresh
}
```

## Rules

- Cache as serialized JSON strings in Hive (not Hive adapters — keeps it simple).
- Models need `toJson()` for caching.
- Always have a `forceRefresh` option.
- Invalidate cache after mutations (create/update/delete).
- Clear caches on logout via `clearAll()`.
- Set reasonable cache expiry if needed (store timestamp with data).
