---
description: Refactor a large feature — split into sub-features, extract shared widgets, move models to core, clean up state fields, and reorganize files while maintaining all existing functionality.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path>"
---

# Refactor Feature

Refactor a feature that has grown too large — split, reorganize, and clean up while preserving functionality.

## Arguments

- `$ARGUMENTS`: Path to the feature to refactor.

## Step 1: Analyze Current State

1. Count files in the feature directory.
2. Count lines in cubit.dart and state.dart.
3. Count methods in the cubit.
4. Count state fields.
5. Count screens in views/.
6. Identify logical groupings of methods/state.

## Step 2: Identify Split Points

A feature should be split when:
- Cubit has > 15 methods
- State has > 10 DataState fields
- Feature has > 5 screens
- Clear sub-domain boundaries exist (e.g., `orders` could split into `order_list`, `order_detail`, `order_create`)

## Step 3: Plan Refactoring

Present the plan to the user:
```
## Refactoring Plan for {feature}

### Current: 1 feature, 20 methods, 12 state fields, 8 screens

### Proposed Split:
1. {feature}_list/ — list view, pagination, search, filters
   - State: itemsData, searchQuery, filters, currentPage
   - Methods: fetchItems, loadMore, search, setFilter

2. {feature}_detail/ — detail view, actions
   - State: itemData, deleteStatus
   - Methods: fetchDetail, deleteItem

3. {feature}_create/ — create/edit form
   - State: createData, formFields
   - Methods: create, update, setFormField

### Shared Extractions:
- {Feature}Model → move to core/models/ (used by all sub-features)
- {Feature}Card widget → move to shared/widgets/
```

## Step 4: Execute Refactoring

For each new sub-feature:
1. Create the full `data/domain/presentation` structure
2. Move relevant state fields from old state
3. Move relevant cubit methods from old cubit
4. Move relevant repository methods
5. Move relevant screens and widgets
6. Update all imports across the codebase
7. Register new cubit in `app_page.dart`
8. Update routes

## Step 5: Clean Up Old Feature

After splitting:
1. Remove migrated methods/fields from original cubit/state
2. If original feature is now empty, delete it
3. Remove old cubit registration from `app_page.dart`
4. Update routes

## Step 6: Verify

1. Run `dart analyze lib/` — zero errors
2. Search for broken imports
3. Verify all routes still work
4. Check no cubit registrations are missing

## Rules

- NEVER lose functionality during refactoring.
- Move, don't copy — avoid code duplication.
- Update ALL imports across the codebase when moving files.
- Ask user to confirm the split plan before executing.
- Commit after each logical step for easy rollback.
- If models are used by multiple sub-features, move to core/models/.
