---
description: Add a custom Dio interceptor to the API service — create interceptor class, register in ApiService's interceptor chain, handle dependencies.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<interceptor-name> e.g. CacheInterceptor, RetryInterceptor"
---

# Add Dio Interceptor

Create a custom Dio interceptor and register it in ApiService.

## Arguments

Parse $ARGUMENTS for the interceptor name. If missing, ask what type:
- **Cache**: Cache GET responses for configurable duration
- **Retry**: Retry failed requests with exponential backoff
- **Connectivity**: Check network before requests
- **Rate Limit**: Throttle requests

## Step 1: Read Existing Interceptors

Read `lib/core/api_service/api_service.dart` and existing interceptors (AuthInterceptor, LoggingInterceptor) to match patterns.

## Step 2: Create Interceptor

Create `lib/core/api_service/<name_snake>.dart`:

```dart
import 'package:dio/dio.dart';

class CacheInterceptor extends Interceptor {
  CacheInterceptor();

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    // Pre-request logic
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    // Post-response logic
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    // Error handling logic
    handler.next(err);
  }
}
```

## Step 3: Register in ApiService

Add to the interceptor chain in ApiService constructor.
Order: Auth first -> Custom interceptors -> Logging last.

## Step 4: Verify

Run `dart analyze lib/` and fix any issues.

## Rules

- Always extend `Interceptor` from Dio
- `handler.next()` passes to next interceptor
- `handler.resolve()` short-circuits with a response
- `handler.reject()` short-circuits with an error
- Interceptor order matters: Auth -> Custom -> Logging
