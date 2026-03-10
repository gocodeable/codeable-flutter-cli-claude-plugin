---
description: Clean all dead code, optimize APK size, and produce an optimized release APK/AAB — removes unused imports, dependencies, colors, assets, widgets, models, endpoints, and localization keys, then builds with all optimization flags.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent, AskUserQuestion
argument-hint: "(no arguments needed)"
---

# Dead Code Cleanup & Optimized Release Build

You are a dead code elimination and release optimization agent for codeable_cli Flutter projects.

## Phase 1: Unused Imports
Run `dart analyze lib/`, collect `unused_import` warnings, remove them, verify.

## Phase 2: Unused Dependencies
Search `lib/` for each `pubspec.yaml` dependency's import pattern. Flag zero-reference deps (except framework deps like `flutter_localizations`, lint packages, Firebase native deps). Ask user before removing.

## Phase 3: Unused Asset Paths
Read `constants/asset_paths.dart`. Search for each constant across `lib/`. Remove zero-reference constants. Flag orphaned asset files for user review.

## Phase 4: Unused Colors
Read `constants/app_colors.dart`. Search for `AppColors.{name}` across `lib/`. Remove zero-reference colors.

## Phase 5: Unused Models
Glob `*_model.dart` and `*_response_model.dart`. Search for each class name. Remove files where ALL classes are unreferenced.

## Phase 6: Unused Widgets & Screens
Glob widgets and views directories. Search for class references. Remove unreferenced files.

## Phase 7: Unused Repository Methods
Read each repository interface. Search for method calls. Remove uncalled methods from both interface and impl.

## Phase 8: Unused Cubit Methods & State Fields
Search for cubit method calls and state field reads. Remove unreferenced ones.

## Phase 9: Unused Endpoints
Read `endpoints.dart`. Search for `Endpoints.{name}`. Remove zero-reference constants.

## Phase 10: Unused Localization Keys
Read ARB files. Search for `context.l10n.{key}` and `Localization.{key}`. Remove zero-reference keys from all ARBs and localization service.

## Phase 11: Unused Enums
Glob enum files. Search for references. Remove unused.

## Phase 12: APK Size Optimization

**Android build config**: Ensure release build type has `isMinifyEnabled = true`, `isShrinkResources = true`, proper ProGuard rules.

**Asset audit**: Flag images > 500KB, suggest WebP conversion, check for bloated SVGs.

**Font audit**: Check for unused font weights.

**Dependency audit**: Flag heavy deps, suggest lighter alternatives.

Ask user before making optimization changes.

## Phase 13: Release Build

1. `flutter clean && flutter pub get`
2. If ARBs modified: `flutter gen-l10n`
3. Ask user which flavor to build.
4. Build:
```bash
flutter build apk --release --flavor {flavor} -t lib/main_{flavor}.dart --split-per-abi --obfuscate --split-debug-info=./debug-info/ --tree-shake-icons
flutter build appbundle --release --flavor {flavor} -t lib/main_{flavor}.dart --obfuscate --split-debug-info=./debug-info/ --tree-shake-icons
```
5. Report output paths and sizes.

## Phase 14: Recursive Pass

After all removals, run one more full pass (Phases 1-11). Removing code creates new dead code. Stop when a full pass finds nothing.

## Rules

- NEVER remove code that IS used. When in doubt, keep it.
- Always verify with `dart analyze` after each phase.
- Search ALL of `lib/` and `test/` thoroughly.
- Do NOT remove: main files, app_page, exports barrel, router files, DI setup, Firebase init.
- Do NOT remove framework contract methods (build, initState, dispose, props, copyWith).
- Use parallel Agent subagents for large-scale searches.
- Ask user before deleting actual asset files from disk.
