---
description: Create an onboarding/walkthrough flow — PageView with dot indicators, skip/next buttons, and completion persistence so it only shows once.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "--pages 3 [--with-animation]"
---

# Add Onboarding Flow

Create a walkthrough/onboarding screen shown only on first app launch.

## Arguments

- `$ARGUMENTS`: Number of pages and options. If not provided, ask.

## Step 1: Create Onboarding Feature

`lib/features/shared/onboarding/`:

### Screen

```dart
class OnboardingScreen extends StatefulWidget {
  const OnboardingScreen({super.key});

  @override
  State<OnboardingScreen> createState() => _OnboardingScreenState();
}

class _OnboardingScreenState extends State<OnboardingScreen> {
  final _pageController = PageController();
  int _currentPage = 0;

  final _pages = const [
    OnboardingPage(
      title: 'Welcome',
      description: 'Discover amazing features',
      imagePath: AssetPaths.onboarding1,
    ),
    OnboardingPage(
      title: 'Explore',
      description: 'Browse and find what you need',
      imagePath: AssetPaths.onboarding2,
    ),
    OnboardingPage(
      title: 'Get Started',
      description: 'Create your account and begin',
      imagePath: AssetPaths.onboarding3,
    ),
  ];

  void _onNext() {
    if (_currentPage < _pages.length - 1) {
      _pageController.nextPage(
        duration: const Duration(milliseconds: 300),
        curve: Curves.easeInOut,
      );
    } else {
      _completeOnboarding();
    }
  }

  void _completeOnboarding() {
    Injector.resolve<AppPreferences>().store('onboarding_complete', true);
    context.go(AppRoutes.login);
  }

  @override
  void dispose() {
    _pageController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Column(
          children: [
            // Skip button
            Align(
              alignment: AlignmentDirectional.topEnd,
              child: TextButton(
                onPressed: _completeOnboarding,
                child: Text('Skip', style: context.b1),
              ),
            ),
            // Pages
            Expanded(
              child: PageView.builder(
                controller: _pageController,
                onPageChanged: (i) => setState(() => _currentPage = i),
                itemCount: _pages.length,
                itemBuilder: (_, i) => _pages[i],
              ),
            ),
            // Dot indicators
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: List.generate(
                _pages.length,
                (i) => AnimatedContainer(
                  duration: const Duration(milliseconds: 200),
                  margin: const EdgeInsets.symmetric(horizontal: 4),
                  width: _currentPage == i ? 24 : 8,
                  height: 8,
                  decoration: BoxDecoration(
                    color: _currentPage == i
                        ? AppColors.primary
                        : AppColors.primary.withOpacity(0.3),
                    borderRadius: BorderRadius.circular(4),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 32),
            // Next/Get Started button
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 24),
              child: MueblyButton(
                text: _currentPage == _pages.length - 1
                    ? 'Get Started'
                    : 'Next',
                onPressed: _onNext,
              ),
            ),
            const SizedBox(height: 32),
          ],
        ),
      ),
    );
  }
}
```

### Onboarding Page Widget
```dart
class OnboardingPage extends StatelessWidget {
  const OnboardingPage({
    super.key,
    required this.title,
    required this.description,
    required this.imagePath,
  });

  final String title;
  final String description;
  final String imagePath;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(24),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Expanded(
            child: imagePath.isSvg
                ? SvgPicture.asset(imagePath)
                : Image.asset(imagePath),
          ),
          const SizedBox(height: 32),
          Text(title, style: context.h1, textAlign: TextAlign.center),
          const SizedBox(height: 16),
          Text(description, style: context.b1, textAlign: TextAlign.center),
        ],
      ),
    );
  }
}
```

## Step 2: Add Route Guard

In GoRouter redirect:
```dart
redirect: (context, state) {
  final prefs = Injector.resolve<AppPreferences>();
  final onboardingDone = prefs.retrieve<bool>('onboarding_complete') ?? false;

  if (!onboardingDone && state.matchedLocation != AppRoutes.onboarding) {
    return AppRoutes.onboarding;
  }
  // ... existing auth redirect
}
```

## Step 3: Wire Route

Add onboarding route and screen export.

## Step 4: Verify

Run `dart analyze lib/` and fix any issues.
