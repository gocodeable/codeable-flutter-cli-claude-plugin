# Codeable Claude Plugin

AI-powered companion for [codeable_cli](https://pub.dev/packages/codeable_cli) — the Flutter Clean Architecture scaffolding tool.

**13 skills, 34 commands, 9 agents, 2 hooks** — everything Claude needs to work expertly with codeable_cli projects.

## Installation

### Via Plugin Marketplace
```
/plugin marketplace add gocodeable/codeable-claude-plugin
/plugin install codeable@codeable
```

### Local Development
```bash
claude --plugin-dir ./codeable-claude-plugin
```

## Skills (13 — Auto-invoked)

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

## Commands (34 — User-invoked)

### Feature Scaffolding
| Command | Description |
|---------|-------------|
| `/codeable:feature` | Scaffold new feature with all layers wired |
| `/codeable:add-screen` | Add screen to existing feature |
| `/codeable:add-bottom-nav` | Bottom navigation with StatefulShellRoute |
| `/codeable:add-auth-flow` | Login, register, forgot password, OTP, route guards |
| `/codeable:add-onboarding` | Walkthrough with PageView, dots, persistence |

### API & Data
| Command | Description |
|---------|-------------|
| `/codeable:add-api` | Wire API endpoint end-to-end (endpoint → model → repo → cubit) |
| `/codeable:add-model` | Generate Dart model from JSON |
| `/codeable:add-cubit-state` | Add state field + cubit method |
| `/codeable:add-cache` | Hive caching with cache-then-network strategy |
| `/codeable:add-socket` | WebSocket real-time connection with auto-reconnect |
| `/codeable:add-image-upload` | Camera/gallery pick, crop, compress, multipart upload |

### UI Patterns
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
| `/codeable:add-deeplink` | Android App Links + iOS Universal Links |
| `/codeable:dark-mode` | Theme switching with persistence |

### Localization & RTL
| Command | Description |
|---------|-------------|
| `/codeable:localize` | Localize hardcoded strings using context.l10n |
| `/codeable:fix-rtl` | Migrate to RTL-safe layout properties |

### Wiring
| Command | Description |
|---------|-------------|
| `/codeable:wire-route` | Wire routing for existing screen |
| `/codeable:wire-di` | Wire DI registration for existing feature |

### Quality & Build
| Command | Description |
|---------|-------------|
| `/codeable:simplify` | Review changed code for quality and fix |
| `/codeable:audit` | Convention compliance check |
| `/codeable:generate-tests` | Generate unit + widget tests |
| `/codeable:dependency-check` | Outdated, unused, vulnerable deps audit |
| `/codeable:refactor-feature` | Split large feature into sub-features |
| `/codeable:release-build` | Dead code cleanup + optimized release APK/AAB |

## Agents (9 — Delegated)

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

## Hooks (2 — Auto-triggered)

| Hook | Trigger | Description |
|------|---------|-------------|
| Post-edit lint | After Write/Edit on `.dart` files | Runs `dart analyze` on edited file |
| Stop summary | When Claude finishes | Shows final analysis summary |

## Requirements

- Claude Code v1.0.33+
- Flutter project generated by `codeable_cli`

## License

MIT
