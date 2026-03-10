---
name: flutter-testing
description: Testing patterns for codeable_cli Clean Architecture projects — unit tests for cubits, repository mocks, widget tests, integration tests, and test organization.
---

# Flutter Testing Patterns (Codeable Style)

## Test Directory Structure

```
test/
├── features/
│   └── {role}/
│       └── {feature}/
│           ├── cubit/
│           │   └── {feature}_cubit_test.dart
│           ├── repository/
│           │   └── {feature}_repository_test.dart
│           └── widgets/
│               └── {widget}_test.dart
├── core/
│   ├── api_service/
│   │   └── api_service_test.dart
│   └── models/
│       └── {model}_test.dart
└── helpers/
    ├── pump_app.dart        # Helper to pump app with providers
    └── mock_repositories.dart  # Shared mock repositories
```

## Cubit Unit Test Pattern

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

// Mock the repository
class MockFeatureRepository extends Mock implements FeatureRepository {}

void main() {
  late FeatureCubit cubit;
  late MockFeatureRepository repository;

  setUp(() {
    repository = MockFeatureRepository();
    cubit = FeatureCubit(repository: repository);
  });

  tearDown(() => cubit.close());

  group('FeatureCubit', () {
    test('initial state is correct', () {
      expect(cubit.state, const FeatureState());
      expect(cubit.state.someData.isInitial, true);
    });

    blocTest<FeatureCubit, FeatureState>(
      'emits [loading, loaded] when fetchData succeeds',
      build: () {
        when(() => repository.fetchData()).thenAnswer(
          (_) async => RepositoryResponse(
            isSuccess: true,
            data: mockData,
          ),
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
      'emits [loading, failure] when fetchData fails',
      build: () {
        when(() => repository.fetchData()).thenAnswer(
          (_) async => RepositoryResponse(
            isSuccess: false,
            message: 'Network error',
          ),
        );
        return cubit;
      },
      act: (cubit) => cubit.fetchData(),
      expect: () => [
        const FeatureState(someData: DataState.loading()),
        const FeatureState(
          someData: DataState.failure(error: 'Network error'),
        ),
      ],
    );
  });
}
```

## Repository Unit Test Pattern

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:dio/dio.dart';

class MockApiService extends Mock implements ApiService {}

void main() {
  late FeatureRepositoryImpl repository;
  late MockApiService apiService;

  setUp(() {
    apiService = MockApiService();
    // Note: Since ApiService is a singleton, you may need to use
    // dependency injection or a test-specific constructor
    repository = FeatureRepositoryImpl();
  });

  group('fetchData', () {
    test('returns success with parsed data', () async {
      when(() => apiService.get(Endpoints.featureData)).thenAnswer(
        (_) async => Response(
          data: {'status': 'success', 'data': {'id': '1', 'name': 'Test'}},
          statusCode: 200,
          requestOptions: RequestOptions(),
        ),
      );

      final result = await repository.fetchData();

      expect(result.isSuccess, true);
      expect(result.data, isNotNull);
      expect(result.data!.id, '1');
    });

    test('returns failure on AppApiException', () async {
      when(() => apiService.get(Endpoints.featureData)).thenThrow(
        AppApiException('Server error', statusCode: 500),
      );

      final result = await repository.fetchData();

      expect(result.isSuccess, false);
      expect(result.message, 'Server error');
    });
  });
}
```

## Model Unit Test Pattern

```dart
void main() {
  group('OrderModel', () {
    test('fromJson creates model correctly', () {
      final json = {
        'id': '123',
        'name': 'Test Order',
        'total': 99.99,
        'items': [
          {'id': '1', 'name': 'Item 1'},
        ],
      };

      final model = OrderModel.fromJson(json);

      expect(model.id, '123');
      expect(model.name, 'Test Order');
      expect(model.total, 99.99);
      expect(model.items.length, 1);
    });

    test('fromJson handles null/missing fields', () {
      final json = <String, dynamic>{};

      final model = OrderModel.fromJson(json);

      expect(model.id, '');
      expect(model.name, '');
      expect(model.total, 0);
      expect(model.items, isEmpty);
    });

    test('toJson produces correct output', () {
      const model = OrderModel(id: '1', name: 'Test', total: 10, items: []);
      final json = model.toJson();

      expect(json['id'], '1');
      expect(json['name'], 'Test');
    });

    test('copyWith creates modified copy', () {
      const original = OrderModel(id: '1', name: 'Old', total: 10, items: []);
      final copy = original.copyWith(name: 'New');

      expect(copy.name, 'New');
      expect(copy.id, '1'); // unchanged
    });

    test('equality works via Equatable', () {
      const a = OrderModel(id: '1', name: 'Test', total: 10, items: []);
      const b = OrderModel(id: '1', name: 'Test', total: 10, items: []);

      expect(a, equals(b));
    });
  });
}
```

## Widget Test Pattern

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:bloc_test/bloc_test.dart';
import 'package:mocktail/mocktail.dart';

class MockFeatureCubit extends MockCubit<FeatureState>
    implements FeatureCubit {}

void main() {
  late MockFeatureCubit cubit;

  setUp(() {
    cubit = MockFeatureCubit();
  });

  Widget buildSubject() {
    return MaterialApp(
      home: BlocProvider<FeatureCubit>.value(
        value: cubit,
        child: const FeatureScreen(),
      ),
    );
  }

  testWidgets('shows loading widget when state is loading', (tester) async {
    when(() => cubit.state).thenReturn(
      const FeatureState(someData: DataState.loading()),
    );

    await tester.pumpWidget(buildSubject());

    expect(find.byType(LoadingWidget), findsOneWidget);
  });

  testWidgets('shows data when state is loaded', (tester) async {
    when(() => cubit.state).thenReturn(
      FeatureState(someData: DataState.loaded(data: mockData)),
    );

    await tester.pumpWidget(buildSubject());

    expect(find.text('Expected Text'), findsOneWidget);
  });

  testWidgets('shows retry widget when state is failure', (tester) async {
    when(() => cubit.state).thenReturn(
      const FeatureState(someData: DataState.failure(error: 'Error')),
    );

    await tester.pumpWidget(buildSubject());

    expect(find.byType(RetryWidget), findsOneWidget);
  });
}
```

## Test Dependencies

Add to `dev_dependencies` in `pubspec.yaml`:
```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  bloc_test: ^9.0.0
  mocktail: ^1.0.0
```

## Running Tests

```bash
flutter test                          # Run all tests
flutter test test/features/           # Run feature tests only
flutter test --coverage               # Generate coverage report
flutter test --coverage && genhtml coverage/lcov.info -o coverage/html
```
