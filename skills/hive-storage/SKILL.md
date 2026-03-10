---
name: hive-storage
description: Hive local storage patterns for codeable_cli projects — BaseStorage, AppPreferences, token management, and local caching.
---

# Hive Local Storage Patterns

## BaseStorage (Abstract)

```dart
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
final appPreferences = AppPreferences();
await appPreferences.init('app-storage');
container.registerSingleton<AppPreferences>(appPreferences);
```
