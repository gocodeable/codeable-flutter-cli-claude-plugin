---
name: error-handling
description: Error handling strategy for codeable_cli projects — API errors, network errors, global error handling, toast notifications, retry patterns, and error boundaries.
---

# Error Handling Patterns

## Error Flow

```
API Request (Dio)
  → DioException caught by ApiService._handleRequest()
    → Has response? → Extract server message from response.data['error']['message']
    → No response? → Map DioExceptionType to user-friendly message
    → Converted to AppApiException(message, statusCode, responseData)
  → Repository catches ONLY AppApiException
    → Returns RepositoryResponse(isSuccess: false, message: e.message)
  → Cubit receives failed response (NO try-catch in cubit)
    → Emits DataState.failure(error: message)
  → UI checks state.isFailure
    → Shows RetryWidget or toast
```

## AppApiException

```dart
class AppApiException implements Exception {
  AppApiException(this.message, {this.statusCode, this.responseData});
  final String message;
  final int? statusCode;
  final dynamic responseData;
}
```

## Network Error Handling (in ApiService._handleDioError)

When `e.response` is null (no server response), errors are mapped by `DioExceptionType`:

| DioExceptionType | User Message |
|------------------|-------------|
| `connectionTimeout`, `sendTimeout`, `receiveTimeout` | "Connection timed out. Please try again." |
| `connectionError` (no WiFi, DNS failure) | "No internet connection. Please check your network and try again." |
| `cancel` | "Request was cancelled." |
| default | "Something went wrong. Please try again." |

When `e.response` exists (server responded), backend error messages pass through as-is.

## Repository Error Handling — `execute()` Pattern

All repository methods use `execute()` for centralized error handling. The `execute()` function is defined in `repository_response.dart` alongside `RepositoryResponse`.

**How `execute()` works:**
- The callback returns `T` directly, not `RepositoryResponse<T>`
- `execute()` wraps the result in `RepositoryResponse(isSuccess: true, data: result)`
- Catches `AppApiException` → returns `RepositoryResponse(isSuccess: false, message: e.message)`
- Catches unexpected errors → logs with `AppLogger.error()` + stack trace, returns `RepositoryResponse(isSuccess: false, message: "Something went wrong")`
- **No manual try-catch in repository methods**

**Void operation:**
```dart
@override
Future<RepositoryResponse<void>> updateProfile(ProfileRequest request) {
  return execute(() async {
    await _apiService.put(Endpoints.profileUpdate, request.toJson());
  });
}
```

**Data operation:**
```dart
@override
Future<RepositoryResponse<ProfileModel>> getProfile() {
  return execute(() async {
    final response = await _apiService.get(Endpoints.profile);
    final data = response.data['data'] as Map<String, dynamic>;
    return ProfileModel.fromJson(data);
  });
}
```

**Business logic failure** (throw `AppApiException` inside the callback):
```dart
@override
Future<RepositoryResponse<ProfileModel>> getProfile() {
  return execute(() async {
    final response = await _apiService.get(Endpoints.profile);
    final parsedResponse = ProfileResponseModel.parseResponse(response);
    final data = parsedResponse.response?.data;
    if (!parsedResponse.isSuccess || data == null) {
      throw AppApiException(parsedResponse.error ?? 'Failed to get profile');
    }
    return data;
  });
}
```

**WHY `execute()`?**
- Eliminates repetitive try-catch boilerplate in every repository method
- Centralizes error handling logic in one place
- ApiService already catches ALL exceptions (DioException + generic) and wraps them into `AppApiException`, so the repository only needs to handle `AppApiException` and unexpected errors — both handled by `execute()`

## Cubit Error Handling

**Cubits do NOT have try-catch blocks.** Error handling belongs in the repository layer.

```dart
// CORRECT — no try-catch
Future<void> fetchData() async {
  emit(state.copyWith(data: const DataState.loading()));
  final result = await repository.fetchData();
  if (result.isSuccess) {
    emit(state.copyWith(data: DataState.loaded(data: result.data)));
  } else {
    emit(state.copyWith(data: DataState.failure(error: result.message)));
  }
}

// WRONG — never do this in a cubit
Future<void> fetchData() async {
  try {
    // ...
  } catch (e) {
    emit(state.copyWith(data: DataState.failure(error: e.toString())));
  }
}
```

## UI Error Display

### RetryWidget (for full-screen errors)
```dart
if (state.data.isFailure) {
  return RetryWidget(
    message: state.data.errorMessage,
    onRetry: () => context.read<Cubit>().fetchData(),
  );
}
```

### Toast (for action errors — submit, delete, etc.)
```dart
BlocConsumer<Cubit, State>(
  listener: (context, state) {
    if (state.submitData.isFailure) {
      ToastHelper.showErrorToast(state.submitData.errorMessage ?? 'Failed');
    }
    if (state.submitData.isLoaded) {
      ToastHelper.showSuccessToast('Saved successfully');
    }
  },
  // ...
)
```

### Logging Rules

- **No success logs** — Don't add `AppLogger.info('X fetched successfully')` after every API call. Only use `AppLogger.error()` for failures.
- Use `ToastHelper` for user-facing success feedback, not logger calls.
- **Error logging is centralized** — `execute()` handles logging for unexpected errors, and ApiService uses `AppLogger.error()` for network/Dio errors. Repository methods do not need manual `AppLogger.error()` calls.
- `AppLogger.error()` is appropriate in global error handlers or truly exceptional situations outside the normal `execute()` flow.

### EmptyStateWidget (for empty results, not errors)
```dart
if (state.data.isLoaded && state.data.isEmpty) {
  return const EmptyStateWidget(message: 'No items found');
}
```

## Token Refresh (401 Handling)

Handled automatically by `AuthInterceptor`:
1. 401 received → triggers refresh
2. Concurrent 401s wait for single refresh
3. New tokens stored in AppPreferences
4. Original request retried with new token
5. If refresh fails → clear auth data, redirect to login

## Error Code Mapping

```dart
String getCustomErrorMessage(String errorCode) {
  // Maps backend error codes to user-friendly messages
  // e.g., 'EMAIL_ALREADY_EXISTS' → 'This email is already registered'
}
```

## Global Error Handling

In `main.dart` or `bootstrap.dart`:
```dart
FlutterError.onError = (details) {
  FirebaseCrashlytics.instance.recordFlutterFatalError(details);
};

PlatformDispatcher.instance.onError = (error, stack) {
  FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
  return true;
};
```
