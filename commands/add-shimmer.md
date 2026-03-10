---
description: Add shimmer loading placeholders to a screen — creates skeleton versions of the screen's content that display while data loads, replacing plain loading spinners.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<screen_file_path>"
---

# Add Shimmer Loading

Replace loading spinners with shimmer skeleton placeholders that match the screen's layout.

## Arguments

- `$ARGUMENTS`: Path to the screen file.

## Step 1: Analyze Screen Layout

Read the screen file and identify the content structure when loaded:
- Is it a list? Card grid? Form? Detail page?
- What's the rough layout of each item?

## Step 2: Create Shimmer Widget

`presentation/widgets/{prefix}_shimmer.dart`:

### For List Screens
```dart
class {Prefix}Shimmer extends StatelessWidget {
  const {Prefix}Shimmer({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      physics: const NeverScrollableScrollPhysics(),
      itemCount: 6,
      padding: const EdgeInsets.all(16),
      itemBuilder: (_, __) => const _ShimmerItem(),
    );
  }
}

class _ShimmerItem extends StatelessWidget {
  const _ShimmerItem();

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 16),
      child: Shimmer.fromColors(
        baseColor: Colors.grey[300]!,
        highlightColor: Colors.grey[100]!,
        child: Row(
          children: [
            // Image placeholder
            Container(
              width: 80,
              height: 80,
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(12),
              ),
            ),
            const SizedBox(width: 12),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Title placeholder
                  Container(
                    height: 16,
                    width: double.infinity,
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(4),
                    ),
                  ),
                  const SizedBox(height: 8),
                  // Subtitle placeholder
                  Container(
                    height: 12,
                    width: 150,
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(4),
                    ),
                  ),
                  const SizedBox(height: 8),
                  // Price placeholder
                  Container(
                    height: 14,
                    width: 80,
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(4),
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### For Grid Screens
```dart
class {Prefix}GridShimmer extends StatelessWidget {
  const {Prefix}GridShimmer({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      physics: const NeverScrollableScrollPhysics(),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        childAspectRatio: 0.7,
        crossAxisSpacing: 12,
        mainAxisSpacing: 12,
      ),
      padding: const EdgeInsets.all(16),
      itemCount: 6,
      itemBuilder: (_, __) => const _GridShimmerItem(),
    );
  }
}
```

### For Detail Screens
```dart
// Mimic the detail layout: large image, title, description blocks
```

## Step 3: Replace Loading State

In the screen's BlocBuilder:
```dart
// Before
if (state.data.isLoading) return const LoadingWidget();

// After
if (state.data.isLoading) return const {Prefix}Shimmer();
```

## Step 4: Check Shimmer Dependency

Ensure `shimmer` package is in `pubspec.yaml`:
```bash
flutter pub add shimmer
```

Or if the project has a custom shimmer widget, use that instead.

## Rules

- Match the shimmer layout to the actual content layout.
- Use `NeverScrollableScrollPhysics` on shimmer lists (prevent scroll during loading).
- Use rounded containers — never sharp rectangles for shimmer blocks.
- Show 4-8 skeleton items (enough to fill the screen).
- Keep shimmer simple — don't over-detail the skeleton.
