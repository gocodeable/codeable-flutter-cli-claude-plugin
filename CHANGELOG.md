# Changelog

## 1.2.2

- Centralized `execute()` pattern for repository error handling — all repository methods now use `execute()` instead of manual try-catch blocks
- Updated skills: `error-handling`, `dio-patterns`, `flutter-bloc-patterns`, `coding-standards`, `codeable-architecture`
- Updated commands: `add-api`, `feature`
- ApiService uses `AppLogger.error()` instead of `debugPrint` for error logging
- Error logging is centralized in `execute()` and ApiService — repositories no longer need manual `AppLogger.error()` calls

## 1.2.1

- Initial tracked release
