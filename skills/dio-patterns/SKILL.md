---
name: dio-patterns
description: Dio HTTP client patterns for codeable_cli projects — ApiService singleton, interceptors, multipart uploads, error handling, token refresh. Use when working with API calls, networking, or authentication.
---

# Dio & ApiService Patterns

You are an expert in the Dio networking patterns used by codeable_cli projects.

## ApiService (Singleton Factory)

```dart
class ApiService {
  factory ApiService() => _instance;
  static final ApiService _instance = ApiService._internal();
  ApiService._internal() {
    _dio = Dio(BaseOptions(
      connectTimeout: const Duration(seconds: 60),
      receiveTimeout: const Duration(seconds: 60),
      headers: {'Content-Type': 'application/json'},
    ));
    _dio.interceptors.addAll([
      AuthInterceptor(),
      LoggingInterceptor(),
    ]);
  }
}
```

**CRITICAL**: ApiService is ALWAYS a singleton. Never inject it via constructor. Access it as:
```dart
final ApiService _apiService = ApiService();
// OR via GetIt if registered:
final ApiService _apiService = Injector.resolve<ApiService>();
```

## Available Methods

| Method | Signature | Use Case |
|--------|-----------|----------|
| `get` | `get(endpoint, {queryParams})` | Fetch data |
| `post` | `post({endpoint, data})` | Create/submit data |
| `put` | `put(endpoint, data)` | Full update |
| `patch` | `patch(endpoint, data)` | Partial update |
| `delete` | `delete(endpoint, {queryParams})` | Remove data |
| `postMultipart` | `postMultipart(endpoint, {data, files, fileMap, filesFieldName})` | Upload files (create) |
| `putMultipart` | `putMultipart(endpoint, {data, files})` | Upload files (update) |
| `patchMultipart` | `patchMultipart(endpoint, data)` | Upload files (partial) |
| `getExternal` | `getExternal(fullUrl, {queryParams})` | Third-party APIs |

## Multipart Upload Patterns

**Single file with data fields:**
```dart
final response = await _apiService.postMultipart(
  Endpoints.uploadProduct,
  data: {'name': name, 'price': price.toString()},
  fileMap: {'image': imageFile},
);
```

**Multiple files:**
```dart
final response = await _apiService.postMultipart(
  Endpoints.uploadGallery,
  files: imageFiles,
  filesFieldName: 'images',
);
```

**File MIME type detection** is automatic based on extension:
- jpg/jpeg/png/gif/webp → `image/*`
- mp4/mov/avi → `video/*`
- pdf → `application/pdf`
- Fallback: `application/octet-stream`

## Error Handling

**AppApiException**:
```dart
class AppApiException implements Exception {
  AppApiException(this.message, {this.statusCode, this.responseData});
  final String message;
  final int? statusCode;
  final dynamic responseData;
}
```

**ApiService._handleDioError** handles ALL error types centrally:
- **Server errors** (response exists): Extracts message from `response.data['error']['message']`, falls back to HTTP status code messages
- **Network errors** (no response): Maps `DioExceptionType` to user-friendly messages:
  - `connectionTimeout`/`sendTimeout`/`receiveTimeout` → "Connection timed out. Please try again."
  - `connectionError` → "No internet connection. Please check your network and try again."
  - `cancel` → "Request was cancelled."
  - default → "Something went wrong. Please try again."

**Error logging in ApiService** uses `AppLogger.error()` (not `debugPrint`). All network-level error logging is centralized in ApiService, so repositories do not need to log errors.

**In repositories, use `execute()` for centralized error handling — no manual try-catch:**
```dart
@override
Future<RepositoryResponse<SomeData>> fetchData() {
  return execute(() async {
    final response = await _apiService.get(Endpoints.someEndpoint);
    final data = response.data['data'] as Map<String, dynamic>;
    return SomeData.fromJson(data);
  });
}
```

`execute()` catches `AppApiException` (returns error message) and unexpected errors (logs + returns "Something went wrong").

**Cubits NEVER have try-catch.** Error handling belongs in the repository layer only.

## Auth Interceptor Flow

1. **On Request**: Adds `Authorization: Bearer $token` from AppPreferences
2. **On 401 Error**: Triggers token refresh
3. **Refresh**: POST to `auth/refresh` with `{refreshToken: ...}`
4. **Token Rotation**: Stores new access + refresh tokens
5. **Retry**: Retries original request with new token
6. **Concurrency**: Concurrent 401s wait for single refresh request

## Response Parsing

```dart
final response = await _apiService.get(Endpoints.someEndpoint);
final result = ResponseModel.fromApiResponse(
  response,
  SomeResponseModel.fromJson,
);

if (result.isSuccess) {
  return RepositoryResponse(isSuccess: true, data: result.response?.data);
}
return RepositoryResponse(isSuccess: false, message: result.error ?? 'Fallback error');
```

## BaseUrl

Dynamic via Remote Config:
```dart
static String get baseUrl => ApiEnvironment.effectiveBaseUrl;
```

The ApiService uses `_getFullUrl(endpoint)` to prepend the base URL.
