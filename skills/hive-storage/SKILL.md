---
name: hive-storage
description: Hive CE local storage patterns for codeable_cli projects — BaseStorage, AppPreferences, token management, and local caching. Uses hive_ce_flutter (community edition), NOT the original hive_flutter.
---

# Hive CE Local Storage Patterns

codeable_cli projects use **Hive CE** (`hive_ce_flutter`), the community edition continuation of Hive v2. NOT the original `hive_flutter` package.

## Dependencies

```yaml
dependencies:
  hive_ce_flutter: ^2.6.0

dev_dependencies:
  hive_ce_generator: ^1.7.0
  build_runner: ^2.4.0
```

**Imports use `hive_ce`** — NOT `hive`:
```dart
import 'package:hive_ce/hive_ce.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';
```

## BaseStorage (Abstract)

```dart
import 'package:hive_ce/hive_ce.dart';

abstract class BaseStorage {
  late Box<dynamic> _box;

  Future<void> init(String boxName) async =>
    _box = await Hive.openBox(boxName);

  Future<void> remove(String key) => _box.delete(key);
  Future<void> removeAll() => _box.clear();
  T? retrieve<T>(String key) => _box.get(key) as T?;
  Future<void> store<T>(String key, T value) => _box.put(key, value);
  bool hasData(String key) => _box.containsKey(key);
  List<dynamic> getAllKeys() => _box.keys.toList();
}
```

## AppPreferences (Extends BaseStorage)

Pre-built key-value methods:

```dart
// Token management
await prefs.setToken(token);
String? token = prefs.getToken();
await prefs.setRefreshToken(refreshToken);
String? refresh = prefs.getRefreshToken();

// User management
await prefs.setUserId(userId);
String? userId = prefs.getUserId();
await prefs.setUserRole(role);
String? role = prefs.getUserRole();

// Locale
prefs.setAppLocale('es');
String? locale = prefs.getAppLocale();

// Cleanup
prefs.clearAuthData();  // Clears token, refresh, userId, role
prefs.clearAll();       // Clears everything
```

## Access Pattern

```dart
// Via GetIt
final prefs = Injector.resolve<AppPreferences>();

// Or direct if registered as singleton
final prefs = GetIt.instance<AppPreferences>();
```

## Adding New Preferences

When you need to store a new value:
1. Add a private key constant: `final String _myKey = 'my_key';`
2. Add getter: `String? getMyValue() => retrieve<String>(_myKey);`
3. Add setter: `Future<void> setMyValue(String value) => store(_myKey, value);`
4. Add to `clearAll()` if it should be cleared on logout

## Initialization

Done in `AppModule.setup()`:
```dart
await Hive.initFlutter(); // From hive_ce_flutter
final appPreferences = AppPreferences();
await appPreferences.init('app-storage');
container.registerSingleton<AppPreferences>(appPreferences);
```

## Hive CE Type Adapters

Hive CE supports the new `@GenerateAdapters` annotation for automatic adapter generation:

```dart
import 'package:hive_ce/hive_ce.dart';

part 'my_model.g.dart';

@GenerateAdapters([AdapterSpec<MyModel>()])
class MyModel {
  final String id;
  final String name;

  MyModel({required this.id, required this.name});
}
```

Then run:
```bash
dart run build_runner build
```

Register adapters:
```dart
Hive.registerAdapter(MyModelAdapter());
// Or use HiveRegistrar for bulk registration
```

## Hive CE vs Original Hive

| Feature | Original `hive` | `hive_ce` |
|---------|-----------------|-----------|
| Package | `hive_flutter` | `hive_ce_flutter` |
| Import | `package:hive/hive.dart` | `package:hive_ce/hive_ce.dart` |
| Maintenance | Abandoned | Actively maintained |
| WASM support | No | Yes |
| Isolate support | No | Yes (`IsolatedHive`) |
| Adapter generation | `@HiveType`/`@HiveField` | `@GenerateAdapters` (simpler) |
| API | v2 API | Same v2 API (drop-in replacement) |

## Rules

- Always import from `hive_ce` — never `hive`.
- Use `hive_ce_flutter` — never `hive_flutter`.
- Use `hive_ce_generator` — never `hive_generator`.
- The Box API is identical to original Hive — `openBox`, `put`, `get`, `delete`, `clear`.
- For simple key-value storage, no adapters needed — store primitives and JSON strings.
- For complex objects, use `@GenerateAdapters` with `build_runner`.
