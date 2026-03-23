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

## API Key Protection (envied)

- NEVER hardcode API keys in Dart code
- Use the `envied` package for compile-time env variable injection with XOR obfuscation
- Per-flavor `.env` files live in a gitignored `env/` folder at project root (`env/.env.development`, `env/.env.staging`, `env/.env.production`)
- Per-flavor env dart files in `lib/config/env/` (`env_dev.dart`, `env_stg.dart`, `env_prod.dart`) use `@Envied` annotations
- A resolver `AppEnv` class in `lib/config/env/app_env.dart` picks the right env based on current flavor
- Obfuscated fields (API keys, secrets) use `@EnviedField(obfuscate: true)` and must be `static final` (not `const`, since obfuscation generates runtime code)
- Non-obfuscated fields (base URLs) use `static const`
- Firebase Remote Config is still used for dynamic values (e.g., dev base URL override)
- Add to `.gitignore`: `env/`, `google-services.json`, `GoogleService-Info.plist`

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

## Build Obfuscation & ProGuard/R8 (Release Builds)

Dart code obfuscation for release builds:
```bash
flutter build apk --obfuscate --split-debug-info=build/debug-info
flutter build appbundle --obfuscate --split-debug-info=build/debug-info
flutter build ipa --obfuscate --split-debug-info=build/debug-info
```

Android release buildType should include:
```groovy
release {
    shrinkResources true
    minifyEnabled true
    proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
}
```

Use a **Makefile** for consistent release builds with obfuscation flags, so developers don't forget to include them:
```makefile
build-apk:
	flutter build apk --flavor production -t lib/main_production.dart --obfuscate --split-debug-info=build/debug-info

build-aab:
	flutter build appbundle --flavor production -t lib/main_production.dart --obfuscate --split-debug-info=build/debug-info

build-ios:
	flutter build ipa --flavor production -t lib/main_production.dart --obfuscate --split-debug-info=build/debug-info
```

ProGuard rules keep Flutter and plugin classes intact.

## File Upload Security

- MIME type detection based on file extension
- Validate file type before upload
- Don't trust client-provided file types on the backend
