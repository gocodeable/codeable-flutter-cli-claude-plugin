---
description: Add a tabbed view to a screen using TabBar + TabBarView or the project's SlidingTab widget — creates tab content widgets and wires state for active tab tracking.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> --tabs 'All,Pending,Completed'"
---

# Add Tab View

Add tabbed content to a screen in a codeable_cli feature.

## Arguments

- `$ARGUMENTS`: Feature path and tab labels.

## Step 1: Check Available Widgets

Check if the project has a `SlidingTab` or similar core widget. If yes, use it. Otherwise, use Flutter's `DefaultTabController`.

## Option A: With SlidingTab (if available)

```dart
class {Prefix}Screen extends StatelessWidget {
  const {Prefix}Screen({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<{Prefix}Cubit, {Prefix}State>(
      builder: (context, state) {
        return Scaffold(
          appBar: mueblyAppBar(context: context, title: 'Orders'),
          body: Column(
            children: [
              SlidingTab(
                tabs: const ['All', 'Pending', 'Completed'],
                selectedIndex: state.selectedTabIndex,
                onTabChanged: (index) =>
                    context.read<{Prefix}Cubit>().setTabIndex(index),
              ),
              Expanded(
                child: _buildTabContent(context, state),
              ),
            ],
          ),
        );
      },
    );
  }

  Widget _buildTabContent(BuildContext context, {Prefix}State state) {
    switch (state.selectedTabIndex) {
      case 0: return const AllItemsTab();
      case 1: return const PendingItemsTab();
      case 2: return const CompletedItemsTab();
      default: return const SizedBox.shrink();
    }
  }
}
```

## Option B: With DefaultTabController

```dart
class {Prefix}Screen extends StatelessWidget {
  const {Prefix}Screen({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Orders'),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'All'),
              Tab(text: 'Pending'),
              Tab(text: 'Completed'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            AllItemsTab(),
            PendingItemsTab(),
            CompletedItemsTab(),
          ],
        ),
      ),
    );
  }
}
```

## Step 2: Add State Field

```dart
const FeatureState({
  this.selectedTabIndex = 0,
});
final int selectedTabIndex;
```

## Step 3: Add Cubit Method

```dart
void setTabIndex(int index) {
  emit(state.copyWith(selectedTabIndex: index));
}
```

## Step 4: Create Tab Content Widgets

Create each tab's content as a separate widget file in `presentation/widgets/`.

## Step 5: Verify

Run `dart analyze lib/` and fix any issues.
