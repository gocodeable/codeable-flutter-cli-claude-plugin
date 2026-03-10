---
description: Add deep link handling to the app — configures Android App Links and iOS Universal Links, adds GoRouter deep link routes, and handles notification-based navigation.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<link_pattern> e.g. /products/:id"
---

# Add Deep Linking

Configure deep links for the app using GoRouter's built-in support.

## Arguments

- `$ARGUMENTS`: Deep link URL pattern(s). If not provided, ask the user.

## Step 1: Detect Configuration

1. Read `pubspec.yaml` for package name.
2. Read `android/app/build.gradle.kts` for application ID.
3. Read `ios/Runner/Info.plist` for bundle ID.
4. Read `go_router/router.dart` for existing route setup.

## Step 2: Android App Links

Add intent filter to `android/app/src/main/AndroidManifest.xml`:

```xml
<activity ...>
  <!-- Deep Links -->
  <intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
      android:scheme="https"
      android:host="your-domain.com"
      android:pathPrefix="/products" />
  </intent-filter>

  <!-- Custom Scheme (for dev/testing) -->
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="myapp" android:host="open" />
  </intent-filter>
</activity>
```

## Step 3: iOS Universal Links

Add to `ios/Runner/Runner.entitlements`:
```xml
<key>com.apple.developer.associated-domains</key>
<array>
  <string>applinks:your-domain.com</string>
</array>
```

## Step 4: GoRouter Configuration

GoRouter handles deep links automatically if routes match:

```dart
static final router = GoRouter(
  initialLocation: AppRoutes.splash,
  routes: [
    // This route handles: https://your-domain.com/products/123
    GoRoute(
      path: '/products/:id',
      name: AppRouteNames.productDetail,
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductDetailScreen(id: id);
      },
    ),
  ],
);
```

## Step 5: Handle Notification Deep Links

Wire notification tap to GoRouter:

```dart
// In app initialization
fcmService.navigationStream.listen((data) {
  final route = data['route'] as String?;
  final id = data['id'] as String?;
  if (route != null) {
    AppRouter.router.push(route);
  }
});
```

## Step 6: Verify

1. Test Android: `adb shell am start -a android.intent.action.VIEW -d "https://your-domain.com/products/123"`
2. Test iOS: `xcrun simctl openurl booted "https://your-domain.com/products/123"`

## Rules

- GoRouter handles deep link parsing automatically — routes must match URL paths.
- Always handle the case where deep link data is missing or invalid.
- Deep links should redirect to login if user is not authenticated.
- Test with both custom scheme (`myapp://`) and HTTPS universal links.
