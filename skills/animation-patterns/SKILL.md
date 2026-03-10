---
name: animation-patterns
description: Flutter animation patterns for codeable_cli projects — implicit animations, explicit animations, page transitions, hero animations, staggered lists, and shimmer loading.
---

# Flutter Animation Patterns

## Implicit Animations (Simple, Preferred)

Use when a property changes and you want it to animate automatically:

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: isExpanded ? 200 : 100,
  height: isExpanded ? 200 : 100,
  color: isSelected ? AppColors.primary : AppColors.surface,
  child: content,
)

AnimatedOpacity(
  duration: const Duration(milliseconds: 200),
  opacity: isVisible ? 1.0 : 0.0,
  child: content,
)

AnimatedSwitcher(
  duration: const Duration(milliseconds: 300),
  child: isLoading
    ? const LoadingWidget(key: ValueKey('loading'))
    : ContentWidget(key: ValueKey('content'), data: data),
)

AnimatedCrossFade(
  duration: const Duration(milliseconds: 300),
  firstChild: const CompactView(),
  secondChild: const ExpandedView(),
  crossFadeState: isExpanded
    ? CrossFadeState.showSecond
    : CrossFadeState.showFirst,
)
```

## Explicit Animations (Complex, Custom)

Use `StatefulWidget` with `SingleTickerProviderStateMixin`:

```dart
class _AnimatedWidgetState extends State<AnimatedWidget>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _fadeAnimation;
  late final Animation<Offset> _slideAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 500),
      vsync: this,
    );
    _fadeAnimation = Tween<double>(begin: 0, end: 1).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeIn),
    );
    _slideAnimation = Tween<Offset>(
      begin: const Offset(0, 0.3),
      end: Offset.zero,
    ).animate(CurvedAnimation(parent: _controller, curve: Curves.easeOut));

    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose(); // CRITICAL: Always dispose
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _fadeAnimation,
      child: SlideTransition(
        position: _slideAnimation,
        child: content,
      ),
    );
  }
}
```

## Staggered List Animation

Animate list items appearing one after another:

```dart
class StaggeredListItem extends StatefulWidget {
  const StaggeredListItem({
    super.key,
    required this.index,
    required this.child,
  });
  final int index;
  final Widget child;

  @override
  State<StaggeredListItem> createState() => _StaggeredListItemState();
}

class _StaggeredListItemState extends State<StaggeredListItem>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 400),
      vsync: this,
    );
    Future.delayed(Duration(milliseconds: widget.index * 100), () {
      if (mounted) _controller.forward();
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _controller,
      child: SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(0, 0.2),
          end: Offset.zero,
        ).animate(CurvedAnimation(
          parent: _controller,
          curve: Curves.easeOut,
        )),
        child: widget.child,
      ),
    );
  }
}
```

## Hero Animations

For shared element transitions between screens:

```dart
// Source screen
Hero(
  tag: 'product-${product.id}',
  child: MueblyCachedImageWidget(imageUrl: product.imageUrl),
)

// Destination screen
Hero(
  tag: 'product-${product.id}',
  child: MueblyCachedImageWidget(imageUrl: product.imageUrl),
)
```

## Page Transitions

Custom GoRouter page transitions:

```dart
GoRoute(
  path: AppRoutes.detail,
  pageBuilder: (context, state) => CustomTransitionPage(
    child: DetailScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return FadeTransition(opacity: animation, child: child);
    },
  ),
),
```

## Shimmer Loading

Use shimmer placeholders while data loads:

```dart
if (state.data.isLoading) {
  return ListView.builder(
    itemCount: 5,
    itemBuilder: (_, __) => const ShimmerListItem(),
  );
}
```

## Rules

- Prefer implicit animations (AnimatedContainer, AnimatedOpacity) for simple state changes.
- Always dispose AnimationControllers.
- Use `mounted` check before calling `forward()` after delays.
- Keep durations between 200-500ms for UI interactions.
- Use `Curves.easeInOut` as default curve.
- Don't animate on every rebuild — use `buildWhen` to limit.
