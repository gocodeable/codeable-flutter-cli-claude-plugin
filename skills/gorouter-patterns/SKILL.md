---
name: gorouter-patterns
description: GoRouter navigation patterns for codeable_cli projects — route setup, nested shell routes, bottom navigation, passing data between screens, deep linking, and route guards.
---

# GoRouter Navigation Patterns

You are an expert in the GoRouter patterns used by codeable_cli projects.

## Route Files

| File | Purpose |
|------|---------|
| `go_router/routes.dart` | Route path constants (`AppRoutes`) and route name constants (`AppRouteNames`) |
| `go_router/router.dart` | `AppRouter` singleton with `GoRouter` instance and route tree |
| `go_router/exports.dart` | Barrel export for all screen imports |

## Route Constants

```dart
class AppRoutes {
  AppRoutes._();
  static const String splash = '/';
  static const String login = '/login';
  static const String customerHomeScreen = '/customer-home';
  static const String customerProductDetail = '/customer-product-detail';
}

class AppRouteNames {
  AppRouteNames._();
  static const String splash = 'splash';
  static const String login = 'login';
  static const String customerHomeScreen = 'customerHomeScreen';
}
```

## Simple Route

```dart
GoRoute(
  name: AppRouteNames.customerHomeScreen,
  path: AppRoutes.customerHomeScreen,
  builder: (context, state) => const CustomerHomeScreen(),
),
```

## Route with Path Parameters

```dart
// Route definition
GoRoute(
  name: AppRouteNames.productDetail,
  path: '${AppRoutes.productDetail}/:id',
  builder: (context, state) => ProductDetailScreen(
    id: state.pathParameters['id']!,
  ),
),

// Navigation
context.push('${AppRoutes.productDetail}/$productId');
```

## Route with Extra Data

```dart
// Route definition
GoRoute(
  name: AppRouteNames.customerProductDetail,
  path: AppRoutes.customerProductDetail,
  builder: (context, state) {
    final product = state.extra! as ProductModel;
    return CustomerProductDetailScreen(product: product);
  },
),

// Navigation
context.push(AppRoutes.customerProductDetail, extra: productModel);
```

## Route with Map Extra

```dart
// Route definition
GoRoute(
  path: AppRoutes.editTeamMember,
  builder: (context, state) {
    final data = state.extra! as Map<String, String?>;
    return EditTeamMemberScreen(
      name: data['name'],
      email: data['email'],
      imageUrl: data['imageUrl'],
    );
  },
),

// Navigation
context.push(AppRoutes.editTeamMember, extra: {
  'name': member.name,
  'email': member.email,
  'imageUrl': member.imageUrl,
});
```

## Bottom Navigation (StatefulShellRoute)

```dart
StatefulShellRoute.indexedStack(
  branches: <StatefulShellBranch>[
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: AppRoutes.customerHomeScreen,
          name: AppRouteNames.customerHomeScreen,
          builder: (context, state) => const CustomerHomeScreen(),
        ),
      ],
    ),
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: AppRoutes.customerWishlistScreen,
          name: AppRouteNames.customerWishlistScreen,
          builder: (context, state) => const CustomerWishlistScreen(),
        ),
      ],
    ),
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: AppRoutes.customerProfileScreen,
          name: AppRouteNames.customerProfileScreen,
          builder: (context, state) => const CustomerProfileScreen(),
        ),
      ],
    ),
  ],
  builder: (context, state, shell) => CustomerBottomNavWidget(shell: shell),
),
```

## Navigation Methods

```dart
// Push (adds to stack, back button returns to previous)
context.push(AppRoutes.productDetail);

// Go (replaces entire stack — for auth flows, role changes)
context.go(AppRoutes.splash);

// Push named (using route names)
context.pushNamed(AppRouteNames.productDetail, pathParameters: {'id': '123'});

// Pop (go back)
context.pop();

// Push replacement (replaces current screen in stack)
context.pushReplacement(AppRoutes.home);
```

## AppRouter Utility Methods

```dart
// Get current route context
final context = AppRouter.appContext;

// Get current location
final location = AppRouter.getCurrentLocation();

// Check if on a specific route
if (AppRouter.isCurrentRoute(AppRouteNames.home)) { ... }
```

## When Adding a New Route

1. Add path to `AppRoutes` in `routes.dart`
2. Add name to `AppRouteNames` in `routes.dart`
3. Add `GoRoute` to the route tree in `router.dart`
4. Add screen export to `exports.dart`
5. If inside a bottom nav, add to the appropriate `StatefulShellBranch`
