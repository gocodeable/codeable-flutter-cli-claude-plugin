---
description: Generate unit tests for a feature — cubit tests with bloc_test, repository mock tests with mocktail, model serialization tests, and widget tests.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<feature_path> [--cubit|--repo|--models|--widgets|--all]"
---

# Generate Tests

Generate comprehensive tests for a codeable_cli feature using **mocktail** (not mockito) and **bloc_test**.

## Arguments

- `$ARGUMENTS`: Feature path and test scope. Default: `--all`.

## Step 1: Read Feature Code

Read the feature's cubit, state, repository (interface + impl), models, and screen/widget files.

## Step 2: Generate Test Files

### Mock Data Constants

At the top of every test file, define mock data constants so they can be reused across tests:

```dart
// -- Mock Data --
const tItemModel = ItemModel(id: '1', name: 'Test Item', price: 9.99);
const tItemModel2 = ItemModel(id: '2', name: 'Test Item 2', price: 19.99);
const tItemList = [tItemModel, tItemModel2];
const tErrorMessage = 'Something went wrong';
```

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

  test('initial state is correct', () {
    expect(cubit.state, const FeatureState());
  });

  group('fetchData', () {
    blocTest<FeatureCubit, FeatureState>(
      'emits [loading, loaded] on success',
      build: () {
        when(() => repository.fetchData()).thenAnswer(
          (_) async => RepositoryResponse(isSuccess: true, data: tItemList),
        );
        return cubit;
      },
      act: (cubit) => cubit.fetchData(),
      expect: () => [
        const FeatureState(someData: DataState.loading()),
        FeatureState(someData: DataState.loaded(data: tItemList)),
      ],
    );

    blocTest<FeatureCubit, FeatureState>(
      'emits [loading, failure] on error',
      build: () {
        when(() => repository.fetchData()).thenAnswer(
          (_) async => RepositoryResponse(isSuccess: false, message: tErrorMessage),
        );
        return cubit;
      },
      act: (cubit) => cubit.fetchData(),
      expect: () => [
        const FeatureState(someData: DataState.loading()),
        const FeatureState(someData: DataState.failure(error: tErrorMessage)),
      ],
    );

    blocTest<FeatureCubit, FeatureState>(
      'emits [loading, loaded] with empty list when no data',
      build: () {
        when(() => repository.fetchData()).thenAnswer(
          (_) async => RepositoryResponse(isSuccess: true, data: <ItemModel>[]),
        );
        return cubit;
      },
      act: (cubit) => cubit.fetchData(),
      expect: () => [
        const FeatureState(someData: DataState.loading()),
        const FeatureState(someData: DataState.loaded(data: [])),
      ],
    );
  });
}
```

### Repository Impl Tests (`test/features/{role}/{feature}/data/{feature}_repository_impl_test.dart`):

Test the repository implementation with a **mocked ApiService**:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

class MockApiService extends Mock implements ApiService {}

void main() {
  late FeatureRepositoryImpl repository;
  late MockApiService apiService;

  setUp(() {
    apiService = MockApiService();
    repository = FeatureRepositoryImpl(apiService: apiService);
  });

  group('fetchData', () {
    test('returns success with parsed data on 200', () async {
      when(() => apiService.get(Endpoints.items)).thenAnswer(
        (_) async => Response(
          data: {'data': [tItemJson]},
          statusCode: 200,
          requestOptions: RequestOptions(),
        ),
      );

      final result = await repository.fetchData();

      expect(result.isSuccess, true);
      expect(result.data, isNotNull);
      expect(result.data!.first.id, tItemModel.id);
    });

    test('returns failure on AppApiException', () async {
      when(() => apiService.get(Endpoints.items)).thenThrow(
        AppApiException(message: tErrorMessage),
      );

      final result = await repository.fetchData();

      expect(result.isSuccess, false);
      expect(result.message, tErrorMessage);
    });
  });
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
- Loading state shows `CircularProgressIndicator` / `LoadingWidget`
- Loaded state shows expected content
- Failure state shows error message / `RetryWidget`
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

- Use `mocktail` for mocking (NEVER `mockito` -- no code generation needed).
- Use `bloc_test` and `blocTest<Cubit, State>()` for cubit tests.
- Mock data constants defined at the **top** of each test file with `t` prefix (e.g., `tItemModel`, `tErrorMessage`).
- Mock repositories when testing cubits, mock ApiService when testing repository impls.
- Test all state transitions (loading -> loaded, loading -> failure).
- Group related tests with `group()`.
- Use `setUp` and `tearDown` for cubit lifecycle.
- Register fallback values for complex types: `registerFallbackValue(...)` in `setUpAll`.
- Every cubit method should have at minimum: success test, failure test, edge case test.
