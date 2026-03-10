---
description: Generate unit tests for a feature — cubit tests with bloc_test, repository mock tests, model serialization tests, and widget tests with mocktail.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> [--cubit|--repo|--models|--widgets|--all]"
---

# Generate Tests

Generate comprehensive tests for a codeable_cli feature.

## Arguments

- `$ARGUMENTS`: Feature path and test scope. Default: `--all`.

## Step 1: Read Feature Code

Read the feature's cubit, state, repository (interface + impl), models, and screen/widget files.

## Step 2: Generate Test Files

### Cubit Tests (`test/features/{role}/{feature}/cubit/{feature}_cubit_test.dart`):

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:{pkg}/utils/helpers/data_state.dart';
import 'package:{pkg}/utils/helpers/repository_response.dart';

class MockRepository extends Mock implements FeatureRepository {}

void main() {
  late FeatureCubit cubit;
  late MockRepository repository;

  setUp(() {
    repository = MockRepository();
    cubit = FeatureCubit(repository: repository);
  });

  tearDown(() => cubit.close());

  test('initial state', () {
    expect(cubit.state, const FeatureState());
  });

  // For EACH cubit method, generate:
  // 1. Success test
  // 2. Failure test
  // 3. Edge case tests (empty data, null fields)

  blocTest<FeatureCubit, FeatureState>(
    'fetchData emits [loading, loaded] on success',
    build: () {
      when(() => repository.fetchData()).thenAnswer(
        (_) async => RepositoryResponse(isSuccess: true, data: mockData),
      );
      return cubit;
    },
    act: (cubit) => cubit.fetchData(),
    expect: () => [
      const FeatureState(someData: DataState.loading()),
      FeatureState(someData: DataState.loaded(data: mockData)),
    ],
  );

  blocTest<FeatureCubit, FeatureState>(
    'fetchData emits [loading, failure] on error',
    build: () {
      when(() => repository.fetchData()).thenAnswer(
        (_) async => RepositoryResponse(isSuccess: false, message: 'Error'),
      );
      return cubit;
    },
    act: (cubit) => cubit.fetchData(),
    expect: () => [
      const FeatureState(someData: DataState.loading()),
      const FeatureState(someData: DataState.failure(error: 'Error')),
    ],
  );
}
```

### Model Tests (`test/features/{role}/{feature}/models/{model}_test.dart`):

For each model, test:
- `fromJson` with complete data
- `fromJson` with null/missing fields (default values)
- `toJson` round-trip
- `copyWith` preserves unchanged fields
- `Equatable` equality

### Widget Tests (`test/features/{role}/{feature}/widgets/{screen}_test.dart`):

For each screen, test:
- Loading state shows `LoadingWidget`
- Loaded state shows expected content
- Failure state shows `RetryWidget`
- Button taps call correct cubit methods
- Form validation (for form screens)

## Step 3: Ensure Dependencies

Check `pubspec.yaml` for test dependencies:
```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  bloc_test: ^9.0.0
  mocktail: ^1.0.0
```

## Step 4: Run Tests

```bash
flutter test test/features/{role}/{feature}/
```

Fix any failures.

## Rules

- Use `mocktail` for mocking (not `mockito`).
- Use `bloc_test` for cubit tests.
- Mock repositories, not ApiService (test cubit logic, not HTTP).
- Test all state transitions (loading → loaded, loading → failure).
- Create mock data constants at the top of test files.
- Group related tests with `group()`.
- Use `setUp` and `tearDown` for cubit lifecycle.
