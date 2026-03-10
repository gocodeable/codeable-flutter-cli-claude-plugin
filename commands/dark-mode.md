---
description: Add dark mode/theme switching support — creates theme data, dark color palette, theme cubit, and wires ThemeMode into MaterialApp with persistence.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "(no arguments needed)"
---

# Add Dark Mode

Add theme switching (light/dark mode) with persistence.

## Step 1: Detect Current Theme Setup

Read `lib/app/view/app_view.dart` for the MaterialApp setup.
Read `lib/constants/app_colors.dart` for current color definitions.

## Step 2: Create Dark Colors

Add dark variants to `AppColors`:

```dart
class AppColors {
  // Light theme (existing)
  static const Color background = Color(0xFFFFFFFF);
  static const Color surface = Color(0xFFF5F5F5);
  static const Color textPrimary = Color(0xFF1A1A1A);
  static const Color textSecondary = Color(0xFF757575);

  // Dark theme
  static const Color darkBackground = Color(0xFF121212);
  static const Color darkSurface = Color(0xFF1E1E1E);
  static const Color darkTextPrimary = Color(0xFFE0E0E0);
  static const Color darkTextSecondary = Color(0xFF9E9E9E);

  // Shared (same in both themes)
  static const Color primary = Color(0xFF4B9A8D);
  static const Color error = Color(0xFFE53935);
  static const Color success = Color(0xFF43A047);
}
```

## Step 3: Create ThemeData

`lib/config/theme/app_theme.dart`:

```dart
class AppTheme {
  static ThemeData get light => ThemeData(
    brightness: Brightness.light,
    scaffoldBackgroundColor: AppColors.background,
    colorScheme: const ColorScheme.light(
      primary: AppColors.primary,
      surface: AppColors.surface,
      error: AppColors.error,
    ),
    appBarTheme: const AppBarTheme(
      backgroundColor: AppColors.background,
      foregroundColor: AppColors.textPrimary,
      elevation: 0,
    ),
    // ... other theme properties
  );

  static ThemeData get dark => ThemeData(
    brightness: Brightness.dark,
    scaffoldBackgroundColor: AppColors.darkBackground,
    colorScheme: const ColorScheme.dark(
      primary: AppColors.primary,
      surface: AppColors.darkSurface,
      error: AppColors.error,
    ),
    appBarTheme: const AppBarTheme(
      backgroundColor: AppColors.darkBackground,
      foregroundColor: AppColors.darkTextPrimary,
      elevation: 0,
    ),
    // ... other theme properties
  );
}
```

## Step 4: Create Theme Cubit

`lib/core/theme/cubit/theme_cubit.dart`:

```dart
class ThemeCubit extends Cubit<ThemeMode> {
  ThemeCubit() : super(ThemeMode.system) {
    _loadTheme();
  }

  final AppPreferences _prefs = Injector.resolve<AppPreferences>();

  void _loadTheme() {
    final saved = _prefs.retrieve<String>('theme_mode');
    if (saved == 'light') emit(ThemeMode.light);
    if (saved == 'dark') emit(ThemeMode.dark);
  }

  void setThemeMode(ThemeMode mode) {
    _prefs.store('theme_mode', mode.name);
    emit(mode);
  }

  void toggleTheme() {
    final newMode = state == ThemeMode.dark ? ThemeMode.light : ThemeMode.dark;
    setThemeMode(newMode);
  }
}
```

## Step 5: Wire into MaterialApp

```dart
BlocBuilder<ThemeCubit, ThemeMode>(
  builder: (context, themeMode) {
    return MaterialApp.router(
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      themeMode: themeMode,
      routerConfig: AppRouter.router,
    );
  },
)
```

## Step 6: Add Theme Toggle UI

In settings screen:
```dart
ListTile(
  title: Text('Dark Mode', style: context.b1),
  trailing: Switch(
    value: context.watch<ThemeCubit>().state == ThemeMode.dark,
    onChanged: (_) => context.read<ThemeCubit>().toggleTheme(),
  ),
)
```

## Step 7: Use Theme-Aware Colors

Replace hardcoded colors with theme colors:
```dart
// Before
color: AppColors.background

// After (theme-aware)
color: Theme.of(context).scaffoldBackgroundColor
// or
color: Theme.of(context).colorScheme.surface
```

## Step 8: Verify

Run `dart analyze lib/` and test both themes visually.
