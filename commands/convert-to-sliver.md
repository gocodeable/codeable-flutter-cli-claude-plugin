---
description: Convert a screen from Column/ListView to CustomScrollView with Slivers — enables collapsing app bars, sticky headers, mixed scroll content, and better scroll performance.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<screen_file_path>"
---

# Convert to Slivers

Convert a screen to use `CustomScrollView` with Slivers for advanced scroll behavior.

## Arguments

- `$ARGUMENTS`: Path to the screen file.

## Step 1: Read Current Screen

Analyze the current layout structure and identify:
- AppBar → `SliverAppBar`
- Fixed headers → `SliverPersistentHeader`
- Lists → `SliverList`
- Grids → `SliverGrid`
- Fixed widgets → `SliverToBoxAdapter`
- Padding → `SliverPadding`

## Step 2: Convert

### Before (Column + ListView)
```dart
Scaffold(
  appBar: AppBar(title: Text('Products')),
  body: Column(
    children: [
      SearchBar(),
      FilterChips(),
      Expanded(
        child: ListView.builder(
          itemCount: items.length,
          itemBuilder: (_, i) => ItemCard(item: items[i]),
        ),
      ),
    ],
  ),
)
```

### After (CustomScrollView + Slivers)
```dart
Scaffold(
  body: CustomScrollView(
    slivers: [
      // Collapsing app bar
      SliverAppBar(
        title: const Text('Products'),
        floating: true,       // Shows on scroll up
        snap: true,           // Snaps into view
        // pinned: true,      // Always visible
        // expandedHeight: 200, // For flexible space
      ),

      // Search bar (fixed, non-scrolling)
      SliverToBoxAdapter(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: SearchBar(),
        ),
      ),

      // Filter chips
      SliverToBoxAdapter(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 16),
          child: FilterChips(),
        ),
      ),

      // Sticky section header
      SliverPersistentHeader(
        pinned: true,
        delegate: _StickyHeaderDelegate(
          child: Container(
            color: AppColors.background,
            padding: const EdgeInsets.all(16),
            child: Text('Results (${items.length})', style: context.b2),
          ),
        ),
      ),

      // Scrollable list
      SliverList.builder(
        itemCount: items.length,
        itemBuilder: (_, i) => ItemCard(item: items[i]),
      ),

      // Or grid
      SliverGrid.builder(
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
        ),
        itemCount: items.length,
        itemBuilder: (_, i) => ItemCard(item: items[i]),
      ),

      // Bottom padding
      const SliverPadding(padding: EdgeInsets.only(bottom: 80)),
    ],
  ),
)
```

### SliverPersistentHeader Delegate
```dart
class _StickyHeaderDelegate extends SliverPersistentHeaderDelegate {
  const _StickyHeaderDelegate({required this.child});
  final Widget child;

  @override
  Widget build(context, shrinkOffset, overlapsContent) => child;

  @override
  double get maxExtent => 50;

  @override
  double get minExtent => 50;

  @override
  bool shouldRebuild(covariant _StickyHeaderDelegate oldDelegate) =>
      child != oldDelegate.child;
}
```

## SliverAppBar Variants

```dart
// Floating — hides on scroll down, shows on scroll up
SliverAppBar(floating: true, snap: true)

// Pinned — always visible, shrinks to toolbar height
SliverAppBar(pinned: true, expandedHeight: 200)

// Collapsing with image
SliverAppBar(
  expandedHeight: 250,
  pinned: true,
  flexibleSpace: FlexibleSpaceBar(
    title: Text('Title'),
    background: Image.network(url, fit: BoxFit.cover),
  ),
)
```

## Step 3: Verify

Run `dart analyze lib/` and fix any issues.

## Rules

- Every child of `CustomScrollView.slivers` must be a Sliver widget.
- Wrap non-sliver widgets with `SliverToBoxAdapter`.
- Use `SliverList.builder` / `SliverGrid.builder` for large lists (not `.list()`).
- Add `SliverPadding` for bottom spacing (especially with bottom nav).
- Keep `SliverAppBar` as the first sliver.
