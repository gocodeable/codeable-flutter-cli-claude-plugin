<p align="center">
  <img src="https://img.shields.io/badge/CLAUDE-PLUGIN-b7ff00?style=for-the-badge&labelColor=0e0926" alt="Claude Plugin" />
  <img src="https://img.shields.io/badge/FLUTTER-352e5c?style=for-the-badge&labelColor=0e0926&logo=flutter&logoColor=b7ff00" alt="Flutter" />
  <img src="https://img.shields.io/badge/DART-352e5c?style=for-the-badge&labelColor=0e0926&logo=dart&logoColor=b7ff00" alt="Dart" />
  <img src="https://img.shields.io/badge/v1.1.0-6b6190?style=for-the-badge&labelColor=0e0926" alt="v1.1.0" />
</p>

<p align="center">
  <strong>AI-powered companion for Codeable Flutter CLI — 13 skills, 38 commands, 9 agents, and 2 hooks for Flutter Clean Architecture projects.</strong>
</p>

---

## Overview

The **Codeable Flutter CLI — Claude Plugin** gives Claude deep knowledge of the [codeable_cli](https://pub.dev/packages/codeable_cli) architecture so it can scaffold features, wire layers, generate tests, and enforce conventions — all through natural conversation.

Built for **Flutter developers** who use Clean Architecture and want Claude to understand their project structure out of the box.

---

## Installation

### Auto-installed

Projects created with `codeable_cli create` include this plugin automatically via `.claude/settings.json`. No setup required — just open the project in Claude Code.

### Manual

```bash
claude plugin add gocodeable/codeable-flutter-cli-claude-plugin
```

---

## Skills (13)

Automatically loaded into Claude's context for deep architecture awareness.

| Skill | What Claude Learns |
|-------|-------------------|
| `codeable-architecture` | Clean Architecture layers, feature structure, data flow, DI, routing |
| `flutter-bloc-patterns` | DataState wrapper, Cubit/State, Repository, BlocBuilder/Consumer |
| `codeable-conventions` | Naming, file organization, imports, styling, core widgets |
| `dio-patterns` | ApiService singleton, interceptors, multipart uploads, token refresh |
| `gorouter-patterns` | Routes, shell routes, bottom nav, data passing, deep linking |
| `hive-storage` | BaseStorage, AppPreferences, token management, caching |
| `flutter-testing` | Unit tests, cubit tests, widget tests with bloc_test + mocktail |
| `helpers-utilities` | ToastHelper, AppLogger, DateTimeHelper, DataState, Permissions |
| `firebase-patterns` | FCM, Remote Config, Analytics, Crashlytics, multi-flavor setup |
| `error-handling` | Error flow, AppApiException, retry patterns, toast notifications |
| `security-patterns` | Secure storage, input validation, SSL, OWASP Mobile Top 10 |
| `animation-patterns` | Implicit/explicit animations, hero, staggered lists, shimmer |
| `accessibility-patterns` | Semantics, touch targets, contrast, screen reader, text scaling |

---

## Commands (38)

### Feature Scaffolding (5)

| Command | Description |
|---------|-------------|
| `/codeable:feature` | Scaffold new feature with all layers wired |
| `/codeable:add-screen` | Add screen to existing feature |
| `/codeable:add-bottom-nav` | Bottom navigation with StatefulShellRoute |
| `/codeable:add-auth-flow` | Login, register, forgot password, OTP, route guards |
| `/codeable:add-onboarding` | Walkthrough with PageView, dots, persistence |

### API & Data (7)

| Command | Description |
|---------|-------------|
| `/codeable:add-api` | Wire API endpoint end-to-end (endpoint → model → repo → cubit) |
| `/codeable:add-model` | Generate Dart model from JSON |
| `/codeable:add-cubit-state` | Add state field + cubit method |
| `/codeable:add-cache` | Hive caching with cache-then-network strategy |
| `/codeable:add-socket` | WebSocket real-time connection with auto-reconnect |
| `/codeable:add-image-upload` | Camera/gallery pick, crop, compress, multipart upload |
| `/codeable:add-hive-model` | Generate Hive TypeAdapter model with box helpers |

### UI Patterns (12)

| Command | Description |
|---------|-------------|
| `/codeable:add-form` | Form screen with validation, controllers, submission |
| `/codeable:add-pagination` | Infinite scroll with page tracking |
| `/codeable:add-search` | Search with debounce (API or local mode) |
| `/codeable:add-filter` | Filter chips, sort dropdown, filter sheet |
| `/codeable:add-tab-view` | Tabbed content with SlidingTab or TabBar |
| `/codeable:add-dialog` | Confirmation, action sheet, or form dialog |
| `/codeable:add-shimmer` | Shimmer skeleton loading placeholders |
| `/codeable:add-core-widget` | New reusable widget in core_widgets |
| `/codeable:extract-widget` | Extract section from build method |
| `/codeable:convert-to-sliver` | Convert to CustomScrollView with Slivers |
| `/codeable:responsive` | Phone + tablet responsive layouts |
| `/codeable:implement-screen` | Implement a screen from a design or spec |

### Firebase & Config (1)

| Command | Description |
|---------|-------------|
| `/codeable:add-firebase-config` | Add Firebase configuration with multi-flavor support |

### Networking (1)

| Command | Description |
|---------|-------------|
| `/codeable:add-interceptor` | Add custom Dio interceptor with request/response hooks |

### Localization & RTL (2)

| Command | Description |
|---------|-------------|
| `/codeable:localize` | Localize hardcoded strings using context.l10n |
| `/codeable:fix-rtl` | Migrate to RTL-safe layout properties |

### Deeplinks & Theming (2)

| Command | Description |
|---------|-------------|
| `/codeable:add-deeplink` | Android App Links + iOS Universal Links |
| `/codeable:dark-mode` | Theme switching with persistence |

### Wiring (2)

| Command | Description |
|---------|-------------|
| `/codeable:wire-route` | Wire routing for existing screen |
| `/codeable:wire-di` | Wire DI registration for existing feature |

### Quality & Build (6)

| Command | Description |
|---------|-------------|
| `/codeable:simplify` | Review changed code for quality and fix |
| `/codeable:audit` | Convention compliance check |
| `/codeable:generate-tests` | Generate unit + widget tests |
| `/codeable:dependency-check` | Outdated, unused, vulnerable deps audit |
| `/codeable:refactor-feature` | Split large feature into sub-features |
| `/codeable:release-build` | Dead code cleanup + optimized release APK/AAB |

---

## Agents (9)

| Agent | Description |
|-------|-------------|
| `code-reviewer` | Convention-aware Flutter code review |
| `dead-code-finder` | Find unused imports, models, widgets, endpoints, colors |
| `feature-planner` | Plan screens, state, APIs, models before coding |
| `test-writer` | Generate cubit, model, widget tests |
| `performance-analyzer` | Find rebuilds, missing const, memory leaks |
| `api-migrator` | Update all layers when API response changes |
| `accessibility-checker` | Audit screens for a11y issues |
| `security-auditor` | OWASP Mobile Top 10 security review |
| `dependency-updater` | Safe dependency updates with breaking change awareness |

---

## Hooks (2)

| Hook | Trigger | Description |
|------|---------|-------------|
| Post-edit lint | After Write/Edit on `.dart` files | Runs `dart analyze` on edited file |
| Stop summary | When Claude finishes | Shows final analysis summary |

---

## Requirements

- Claude Code v1.0.33+
- Flutter project generated by `codeable_cli`

---

## License

MIT

<p align="center">
  <br />
  <a href="https://gocodeable.com">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="assets/codeable_wordmark_white.svg" />
      <source media="(prefers-color-scheme: light)" srcset="assets/codeable_wordmark.svg" />
      <img src="assets/codeable_wordmark.svg" alt="Codeable" width="140" />
    </picture>
  </a>
  <br />
  <sub>Built by <a href="https://gocodeable.com">Codeable</a></sub>
</p>
