---
name: firebase-patterns
description: Firebase integration patterns for codeable_cli projects — FCM notifications, Remote Config, Analytics, Crashlytics, Cloud Storage, and multi-flavor Firebase setup.
---

# Firebase Patterns

## Multi-Flavor Firebase Setup

codeable_cli projects use per-flavor Firebase configs:
```
firebase/
├── development/
│   ├── android/google-services.json
│   └── ios/GoogleService-Info.plist
├── staging/
│   ├── android/google-services.json
│   └── ios/GoogleService-Info.plist
└── production/
    ├── android/google-services.json
    └── ios/GoogleService-Info.plist
```

Firebase is initialized in each `main_{flavor}.dart`:
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  // ...
}
```

## FCM Notifications

**FirebaseNotificationService** (singleton):
```dart
final fcmService = FirebaseNotificationService();

// Get token for backend registration
final token = await fcmService.getFcmToken();

// Listen for navigation from tapped notifications
fcmService.navigationStream.listen((data) {
  // Handle deep link from notification payload
});
```

**Three notification handlers:**
1. **Foreground** (`onMessage`): Shows local notification immediately
2. **Background** (`onBackgroundMessage`): Full Firebase init, shows local notification
3. **Tap** (`onMessageOpenedApp`): Adds payload to `navigationStream` for navigation

**LocalNotificationService**:
```dart
final localNotif = LocalNotificationService();
await localNotif.initializeLocalNotifications();
await localNotif.sendLocalNotification(title, body, payload);
```

## Remote Config

```dart
final remoteConfig = Injector.resolve<RemoteConfigService>();

// Typically used for:
// - Dynamic base URL
// - Feature flags
// - App version checks
// - Maintenance mode
```

Access pattern:
```dart
static String get effectiveBaseUrl => RemoteConfigService.instance.getString('base_url');
```

## Analytics (if integrated)

```dart
await FirebaseAnalytics.instance.logEvent(
  name: 'screen_view',
  parameters: {'screen_name': 'home'},
);

await FirebaseAnalytics.instance.setUserProperty(
  name: 'role',
  value: 'customer',
);
```

## Crashlytics (if integrated)

```dart
// Record non-fatal errors
FirebaseCrashlytics.instance.recordError(error, stackTrace);

// Set user context
FirebaseCrashlytics.instance.setUserIdentifier(userId);

// Custom keys
FirebaseCrashlytics.instance.setCustomKey('feature', 'orders');
```

## Cloud Storage (if integrated)

```dart
final ref = FirebaseStorage.instance.ref('uploads/$userId/$fileName');
final task = ref.putFile(file);
final url = await (await task).ref.getDownloadURL();
```

## Initialization Order (in AppModule)

1. `Firebase.initializeApp()`
2. `RemoteConfigService` — fetch remote values
3. `ApiService` — uses remote config for base URL
4. `FirebaseNotificationService` — FCM token + handlers
5. `LocalNotificationService` — local notification channels
