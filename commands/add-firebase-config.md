---
description: Configure Firebase for all three flavors (development, staging, production) using flutterfire configure — asks for Firebase project ID, detects bundle IDs from existing config.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: ""
---

# Add Firebase Configuration

Set up Firebase for all three app flavors using `flutterfire configure`.

## Step 1: Gather Information

1. **ASK the user for their Firebase project ID** (e.g., `my-app-12345`). Do NOT proceed without it.
2. Read `pubspec.yaml` for project name.
3. Read `android/app/build.gradle.kts` to detect the applicationId/org name.
4. Detect the three flavors: development (.dev), staging (.stg), production (no suffix).

## Step 2: Verify FlutterFire CLI

Check if `flutterfire` is available. If not, instruct user:
```bash
dart pub global activate flutterfire_cli
```

## Step 3: Run flutterfire configure (3 times)

Based on detected org (e.g., `com.example.myapp`):

### Development:
```bash
flutterfire configure \
  --project=<firebase-project-id> \
  --out=lib/firebase_options_dev.dart \
  --ios-bundle-id=<org>.dev \
  --android-app-id=<org>.dev \
  --yes
```

### Staging:
```bash
flutterfire configure \
  --project=<firebase-project-id> \
  --out=lib/firebase_options_stg.dart \
  --ios-bundle-id=<org>.stg \
  --android-app-id=<org>.stg \
  --yes
```

### Production:
```bash
flutterfire configure \
  --project=<firebase-project-id> \
  --out=lib/firebase_options_prod.dart \
  --ios-bundle-id=<org> \
  --android-app-id=<org> \
  --yes
```

## Step 4: Update Entry Points

Update each main_*.dart to import the correct firebase_options file:
- `main_development.dart` -> `import 'firebase_options_dev.dart';`
- `main_staging.dart` -> `import 'firebase_options_stg.dart';`
- `main_production.dart` -> `import 'firebase_options_prod.dart';`

Ensure `Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform)` is called.

## Step 5: Check Dependencies

Ensure `firebase_core` is in pubspec.yaml dependencies.

## Step 6: Verify

Run `flutter analyze --no-pub` and fix issues.

## Rules

- ALWAYS ask for Firebase project ID first
- Three flavors: development (.dev), staging (.stg), production (no suffix)
- Each flavor gets its own firebase_options_*.dart file
- Use --yes flag to skip interactive prompts
