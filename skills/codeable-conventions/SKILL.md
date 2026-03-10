---
name: codeable-conventions
description: Naming conventions, file organization, import patterns, styling system, and coding standards for codeable_cli Flutter projects. Use when creating new files, naming classes, or following project conventions.
---

# Codeable Conventions

You are an expert in the conventions used by codeable_cli-generated Flutter projects. Apply these strictly when writing or reviewing code.

## Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Files | `snake_case` | `customer_home_screen.dart` |
| Classes | `PascalCase` | `CustomerHomeScreen` |
| Feature widgets | `{role}_{feature}_{purpose}.dart` | `customer_home_appbar.dart` |
| Cubit files | Always `cubit.dart` and `state.dart` | Never `customer_home_cubit.dart` |
| Repository interface | `{role}_{feature}_repository.dart` | `customer_home_repository.dart` |
| Repository impl | `{role}_{feature}_repository_impl.dart` | `customer_home_repository_impl.dart` |
| Response models | `{Name}ResponseModel` | `OrdersResponseModel` |
| Data wrappers | `{Name}Data` | `OrdersData` |
| Entity models | `{Name}Model` | `OrderModel` |
| Request models | `{Name}Request` or `Create{Name}Request` | `CreateOrderRequest` |
| Endpoints | `static const String camelCase` | `static const String customerOrders = 'customer/orders';` |
| Routes | `AppRoutes.{role}{Feature}Screen` | `AppRoutes.customerHomeScreen` |
| Route names | `AppRouteNames.{role}{Feature}Screen` | `AppRouteNames.customerHomeScreen` |

## File Organization

- **One widget class per file** — if a build method gets long, extract into `widgets/` folder
- **Screens** go in `presentation/views/`
- **Extracted widgets** go in `presentation/widgets/`
- **Feature-specific models** go in `data/models/`
- **Shared/reusable models** go in `core/models/{domain}_models/`

## Import Pattern

Use the barrel export for common imports:
```dart
import 'package:{project_name}/exports.dart';
```

This provides: Flutter Material, flutter_bloc, flutter_svg, go_router, AppColors, AppTextStyle, AssetPaths, all core widgets, router exports.

Import feature-specific files directly:
```dart
import 'package:{project_name}/features/{role}/{feature}/presentation/cubit/cubit.dart';
import 'package:{project_name}/features/{role}/{feature}/presentation/cubit/state.dart';
```

## Styling System

**Colors**: `AppColors.xxx` — defined in `constants/app_colors.dart`

**Text styles via context extensions**:
- Headings: `context.h1`, `context.h2`
- Titles: `context.t1`
- Body: `context.b1`, `context.b2`, `context.b3`
- Labels: `context.l1`, `context.l2`, `context.l3`
- With color modifiers: `context.b1.secondary`, `context.l2.white`

**Assets**: Referenced via `AssetPaths.xxx` — defined in `constants/asset_paths.dart`

## Core Widgets (use these, don't reinvent)

All available via `exports.dart`:

| Widget | Usage |
|---|---|
| `MueblyAppBar()` / custom `appBar()` | Standard app bar |
| `MueblyButton` | Primary button (filled, loading state, icons) |
| `MueblyTextField` | Text input (email, password, date, time) |
| `MueblyImagePicker` | Circular image picker |
| `MueblyCachedImageWidget` | Network image with shimmer + error fallback |
| `ImagePickerContainer` | Image picker with camera/gallery sheet + cropping |
| `LoadingWidget` | Loading spinner |
| `RetryWidget` | Error state with retry |
| `EmptyStateWidget` | Empty state placeholder |
| `MueblyDialog` | Modal dialog |
| `MueblyBottomSheet` | Bottom sheet container |

Note: Widget names may use a project-specific prefix (e.g., `Muebly`, `Custom`). Check the project's `utils/widgets/core_widgets/export.dart` for actual names.

## Localization

- English + Spanish (or other languages) via ARB files
- ARB files in `l10n/arb/` (`app_en.arb`, `app_es.arb`)
- In widgets: `context.l10n.keyName`
- In non-widget files: `Localization.keyName`
- Import: `package:{project}/l10n/l10n.dart` for `context.l10n`
- Import: `package:{project}/l10n/localization_service.dart` for static `Localization.xxx`

## Code Style

- Write clean, minimal code. No unnecessary abstractions.
- Use `const` constructors wherever possible.
- Prefer `context.read<Cubit>()` for one-time reads, `context.watch<Cubit>()` for reactive rebuilds.
- Use `BlocBuilder` in views, `BlocConsumer` when you need listener + builder.
- Keep widgets small — extract into separate files.
- Don't add docstrings, comments, or type annotations unless explicitly asked.
- Don't add error handling beyond what's needed.
- Don't refactor or "improve" code beyond the current task.

## Key Workflow Patterns

1. **Adding a screen to existing feature**: Add view in `views/`, add route, add state fields to existing `state.dart`, add methods to existing `cubit.dart`.
2. **Adding a new feature**: Create full `data/domain/presentation` structure. ONE cubit, ONE state. Register cubit in `app_page.dart`. Register repository in DI.
3. **Adding an API endpoint**: Add endpoint constant, create models, update repository interface + impl, add state field, add cubit method.
4. **Extracting widgets**: Move sections from screen's build method into separate files in `widgets/`.
