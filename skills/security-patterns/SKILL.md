---
name: security-patterns
description: Security best practices for codeable_cli Flutter projects — secure storage, API key handling, SSL pinning, input validation, and OWASP mobile top 10 mitigations.
---

# Security Patterns

## Secure Token Storage

Tokens stored in Hive (AppPreferences). For sensitive data, consider `flutter_secure_storage`:
```dart
// Current pattern (Hive)
await prefs.setToken(token);
await prefs.setRefreshToken(refreshToken);

// For higher security (if needed)
final secureStorage = FlutterSecureStorage();
await secureStorage.write(key: 'token', value: token);
```

## API Key Protection

- NEVER hardcode API keys in Dart code
- Use environment variables or build configs:
```dart
// In build config / flavor-specific files
const apiKey = String.fromEnvironment('API_KEY');
```
- Use Firebase Remote Config for dynamic keys
- Add to `.gitignore`: `.env`, `google-services.json`, `GoogleService-Info.plist`

## Input Validation

Always validate at the UI boundary:
```dart
MueblyTextField(
  controller: emailController,
  type: MueblyTextFieldType.email,
  validator: (value) {
    if (value == null || value.isEmpty) return 'Email required';
    if (!RegExp(r'^[^@]+@[^@]+\.[^@]+$').hasMatch(value)) return 'Invalid email';
    return null;
  },
)
```

Common validators (from `field_validators.dart`):
- Email format
- Password strength
- Phone number
- Required fields

## SQL/NoSQL Injection Prevention

- Use parameterized queries (never string concatenation)
- ApiService already serializes data safely via Dio
- Never pass raw user input to endpoint URLs:
```dart
// BAD
_apiService.get('users/$userInput')

// GOOD
_apiService.get('users/${Uri.encodeComponent(userInput)}')
```

## XSS Prevention in WebView

```dart
WebView(
  // Don't enable JavaScript unless needed
  javascriptMode: JavascriptMode.disabled,
  // Restrict navigation
  navigationDelegate: (request) {
    if (request.url.startsWith('https://trusted-domain.com')) {
      return NavigationDecision.navigate;
    }
    return NavigationDecision.prevent;
  },
)
```

## SSL Pinning (if needed)

```dart
// Using dio_http2_adapter or custom SecurityContext
_dio.httpClientAdapter = IOHttpClientAdapter(
  createHttpClient: () {
    final client = HttpClient();
    client.badCertificateCallback = (cert, host, port) {
      // Verify certificate fingerprint
      return cert.sha256Fingerprint == expectedFingerprint;
    };
    return client;
  },
);
```

## Sensitive Data in Logs

The `LoggingInterceptor` only logs in debug mode (`kDebugMode`). In release:
- No request/response bodies logged
- No tokens logged
- No PII in crash reports

## Auth Security

- Token refresh with rotation (new refresh token each time)
- Concurrency protection on refresh (single refresh request)
- Clear all auth data on refresh failure
- Automatic redirect to login on auth failure

## ProGuard/R8 (Release Builds)

Code obfuscation enabled via:
```bash
flutter build apk --obfuscate --split-debug-info=./debug-info/
```

ProGuard rules keep Flutter and plugin classes intact.

## File Upload Security

- MIME type detection based on file extension
- Validate file type before upload
- Don't trust client-provided file types on the backend
