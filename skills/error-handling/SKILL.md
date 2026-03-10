---
name: error-handling
description: Error handling strategy for codeable_cli projects — API errors, global error handling, toast notifications, retry patterns, and error boundaries.
---

# Error Handling Patterns

## Error Flow

```
API Request (Dio)
  → DioException caught by ApiService._handleRequest()
    → Converted to AppApiException(message, statusCode, responseData)
  → Repository catches AppApiException
    → Returns RepositoryResponse(isSuccess: false, message: ...)
  → Cubit receives failed response
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

**Error message extraction from API responses:**
1. Try `response.data['error']['message']` (structured error)
2. Fall back to HTTP status messages:
   - 400: Bad request
   - 401: Unauthorized (triggers token refresh)
   - 403: Forbidden
   - 404: Not found
   - 500: Internal server error
3. Final fallback: DioException message

## Repository Error Handling

ALWAYS catch `AppApiException` — NEVER catch bare `Exception`:
```dart
try {
  final response = await _apiService.get(Endpoints.data);
  // ... parse
} on AppApiException catch (e) {
  return RepositoryResponse(isSuccess: false, message: e.message);
}
```

Use the helper for extraction:
```dart
String extractApiErrorMessage(Object e, String fallback) =>
    e is AppApiException ? e.message : fallback;
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

## Connectivity Handling

Check connectivity before API calls or show offline banner:
```dart
// Using connectivity_plus package
final result = await Connectivity().checkConnectivity();
if (result == ConnectivityResult.none) {
  ToastHelper.showErrorToast('No internet connection');
  return;
}
```
