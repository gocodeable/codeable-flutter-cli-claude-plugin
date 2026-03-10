---
description: Check for outdated, vulnerable, or unnecessary dependencies — runs flutter pub outdated, checks for known vulnerabilities, and suggests lighter alternatives.
allowed-tools: Read, Bash, Glob, Grep
argument-hint: "(no arguments needed)"
---

# Dependency Health Check

Audit project dependencies for updates, security, and bloat.

## Step 1: Check Outdated Packages

```bash
flutter pub outdated
```

Categorize into:
- **Patch updates** (safe to update immediately)
- **Minor updates** (safe, may add new features)
- **Major updates** (may have breaking changes — flag for review)

## Step 2: Unused Dependencies

Read `pubspec.yaml`. For each dependency, search for its import in `lib/`:
```bash
grep -r "import 'package:{dep_name}" lib/
```

Flag packages with zero imports (check exceptions: flutter_localizations, lint packages, build_runner, Firebase native packages).

## Step 3: Duplicate Functionality

Flag packages that overlap:
- Multiple HTTP clients (dio + http)
- Multiple state management (bloc + provider + riverpod)
- Multiple image caching packages
- Multiple date/time formatting packages

## Step 4: Heavy Dependencies

Flag large packages and suggest lighter alternatives where possible.

## Step 5: Security Check

Check for known vulnerabilities:
```bash
# Check pub.dev advisories
flutter pub get --enforce-lockfile
```

## Output

```
## Dependency Health Report

### Outdated (X packages)
| Package | Current | Latest | Type |
|---------|---------|--------|------|
| dio | 5.3.0 | 5.4.0 | minor |

### Unused (X packages)
- [package]: not imported anywhere

### Duplicate Functionality
- [overlapping packages]

### Heavy Dependencies
- [large packages with size estimates]

### Security
- [known vulnerabilities if any]

### Recommendations
1. [prioritized action items]
```
