---
description: Set up a bottom navigation bar with StatefulShellRoute for a role — creates the nav widget, shell route branches, and wires up the tab screens with tab persistence.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<role> --tabs 'home,orders,profile'"
---

# Add Bottom Navigation

Set up a bottom navigation bar using GoRouter's `StatefulShellRoute.indexedStack` for a role with tab persistence.

## Arguments

- `$ARGUMENTS`: Role name and tab names.

If not provided, ask:
1. Which role? (customer, admin, business)
2. What tabs? (comma-separated names like `home,orders,profile`)
3. Icons for each tab?

## Step 1: Create Bottom Nav Widget

`lib/features/{role}/shared/widgets/{role}_bottom_nav.dart`:

```dart
import 'package:{pkg}/exports.dart';

class {Role}BottomNav extends StatelessWidget {
  const {Role}BottomNav({super.key, required this.shell});

  final StatefulNavigationShell shell;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: shell,
      bottomNavigationBar: NavigationBar(
        selectedIndex: shell.currentIndex,
        onDestinationSelected: (index) => shell.goBranch(
          index,
          initialLocation: index == shell.currentIndex,
        ),
        destinations: [
          NavigationDestination(
            icon: const Icon(Icons.home_outlined),
            selectedIcon: const Icon(Icons.home),
            label: context.l10n.home,
          ),
          NavigationDestination(
            icon: const Icon(Icons.shopping_bag_outlined),
            selectedIcon: const Icon(Icons.shopping_bag),
            label: context.l10n.orders,
          ),
          NavigationDestination(
            icon: const Icon(Icons.person_outline),
            selectedIcon: const Icon(Icons.person),
            label: context.l10n.profile,
          ),
        ],
      ),
    );
  }
}
```

## Step 2: Update Router

Add `StatefulShellRoute.indexedStack` to `router.dart`. This pattern preserves tab state when switching between tabs:

```dart
StatefulShellRoute.indexedStack(
  branches: <StatefulShellBranch>[
    // Tab 1: Home
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: AppRoutes.{role}HomeScreen,
          name: AppRouteNames.{role}HomeScreen,
          builder: (context, state) => const {Role}HomeScreen(),
        ),
      ],
    ),
    // Tab 2: Orders
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: AppRoutes.{role}OrdersScreen,
          name: AppRouteNames.{role}OrdersScreen,
          builder: (context, state) => const {Role}OrdersScreen(),
        ),
      ],
    ),
    // Tab 3: Profile
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: AppRoutes.{role}ProfileScreen,
          name: AppRouteNames.{role}ProfileScreen,
          builder: (context, state) => const {Role}ProfileScreen(),
        ),
      ],
    ),
  ],
  builder: (context, state, shell) => {Role}BottomNav(shell: shell),
),
```

## Step 3: Add Routes & Exports

Add route constants for each tab screen. Add screen exports.

## Step 4: Use EdgeInsetsDirectional

Ensure any padding or margin in the bottom nav or tab screens uses `EdgeInsetsDirectional` instead of `EdgeInsets.only(left:)` / `EdgeInsets.only(right:)` for RTL support:
```dart
// Good
padding: const EdgeInsetsDirectional.only(start: 16, end: 16)

// Bad - do NOT use
padding: const EdgeInsets.only(left: 16, right: 16)
```

## Step 5: Verify

Run `dart analyze lib/` and fix any issues.

## Rules

- Use `StatefulShellRoute.indexedStack` for tab persistence -- each tab's state is preserved when switching.
- Use `context.l10n.tabName` for all tab labels (never hardcoded strings).
- Use `EdgeInsetsDirectional` for RTL support in any padding/margin.
- Sub-routes within a tab branch will show inside the same tab (no nav bar flicker).
- Use `context.push()` for screens that should appear OVER the nav bar.
- Use `shell.goBranch(index, initialLocation: true)` to reset tab to initial route on re-tap.
