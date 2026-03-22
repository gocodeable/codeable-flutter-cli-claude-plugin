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

## Repository Error Handling

**ONLY catch `AppApiException`** — no generic `catch (e)`:
```dart
try {
  final response = await _apiService.get(Endpoints.data);
  final result = ResponseModel.fromApiResponse(response, DataResponseModel.fromJson);
  if (result.isSuccess) {
    return RepositoryResponse(isSuccess: true, data: result.response?.data);
  }
  return RepositoryResponse(isSuccess: false, message: result.error ?? 'Failed to fetch data');
} on AppApiException catch (e, s) {
  AppLogger.error('Failed to fetch data', e, s);
  return RepositoryResponse(isSuccess: false, message: e.message);
}
```

**WHY no generic catch?** ApiService._handleRequest() already catches ALL exceptions (DioException + generic) and wraps them into AppApiException. The repository never sees raw exceptions.

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
- `AppLogger.error()` is appropriate in repository catch blocks or global error handlers.
- **Always include stack trace** in repository catch blocks: `on AppApiException catch (e, s)` then `AppLogger.error('descriptive message', e, s)`.

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
