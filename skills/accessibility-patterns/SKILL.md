---
name: accessibility-patterns
description: Accessibility (a11y) best practices for Flutter — Semantics, screen reader support, touch targets, contrast ratios, and dynamic text scaling.
---

# Accessibility Patterns

## Semantics

Add meaning for screen readers:

```dart
Semantics(
  label: 'Product: Wooden Chair, Price: \$99',
  button: true,
  child: ProductCard(product: product),
)

// For images
Semantics(
  label: 'Room redesign preview',
  image: true,
  child: MueblyCachedImageWidget(imageUrl: url),
)

// Exclude decorative elements
ExcludeSemantics(
  child: DecorativeBackground(),
)
```

## Touch Targets

Minimum 48x48dp touch targets (Material guidelines):

```dart
// BAD - too small
GestureDetector(
  onTap: onTap,
  child: Icon(Icons.close, size: 16),
)

// GOOD - adequate touch target
IconButton(
  onPressed: onTap,
  icon: Icon(Icons.close, size: 16),
  // IconButton already ensures 48x48 touch target
)

// For custom widgets
SizedBox(
  width: 48,
  height: 48,
  child: GestureDetector(
    onTap: onTap,
    child: Center(child: Icon(Icons.close, size: 16)),
  ),
)
```

## Text Scaling

Support dynamic text sizing:

```dart
// DON'T constrain text scaling too tightly
// DO test with large text (Settings > Accessibility > Large Text)

// If layout breaks with large text, use:
MediaQuery.withClampedTextScaling(
  minScaleFactor: 1.0,
  maxScaleFactor: 1.5,  // Allow up to 150% but prevent extreme
  child: widget,
)
```

## Color Contrast

Ensure sufficient contrast ratios (WCAG AA):
- Normal text: 4.5:1 minimum
- Large text (18sp+ or 14sp+ bold): 3:1 minimum

```dart
// Check your color combinations
// AppColors.textPrimary on AppColors.background should meet 4.5:1
// Don't rely on color alone to convey information — add icons or text
```

## Focus & Navigation

Support keyboard/switch navigation:

```dart
// TextFields, buttons, and interactive widgets get focus automatically
// For custom interactive widgets:
Focus(
  child: GestureDetector(
    onTap: onTap,
    child: widget,
  ),
)

// Set focus order
FocusTraversalGroup(
  policy: OrderedTraversalPolicy(),
  child: Column(
    children: [
      FocusTraversalOrder(order: NumericFocusOrder(1), child: field1),
      FocusTraversalOrder(order: NumericFocusOrder(2), child: field2),
      FocusTraversalOrder(order: NumericFocusOrder(3), child: submitButton),
    ],
  ),
)
```

## Screen Reader Testing

```bash
# iOS: Settings > Accessibility > VoiceOver
# Android: Settings > Accessibility > TalkBack
```

## Checklist

- [ ] All images have semantic labels
- [ ] All buttons have labels (text or Semantics)
- [ ] Touch targets are at least 48x48dp
- [ ] Color contrast meets WCAG AA (4.5:1)
- [ ] App works with text scaling up to 200%
- [ ] Form fields have labels and error messages accessible
- [ ] Loading/error states announced to screen readers
- [ ] No information conveyed by color alone
