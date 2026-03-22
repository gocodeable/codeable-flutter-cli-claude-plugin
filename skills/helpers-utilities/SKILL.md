---
name: helpers-utilities
description: Built-in helper utilities in codeable_cli projects — ToastHelper, AppLogger, DateTimeHelper, string extensions, responsive helpers, and validators.
---

# Helpers & Utilities

## ToastHelper

```dart
import 'package:{pkg}/utils/helpers/toast_helper.dart';

ToastHelper.showErrorToast('Something went wrong');
ToastHelper.showSuccessToast('Saved successfully');
ToastHelper.showInfoToast('Processing...');
ToastHelper.showCustomToast('Custom message');
```
- Positioned at top center
- Auto-close after 2-4 seconds
- Uses toastification package with SVG icons

## AppLogger

```dart
import 'package:{pkg}/utils/helpers/logger_helper.dart';

AppLogger.error('API failed', exception, stackTrace);
AppLogger.debug('Debug info');
AppLogger.warning('Warning message');
AppLogger.verbose('Verbose details');
```
- Only logs in debug mode
- **No success logs** — Don't add `AppLogger.info('X fetched successfully')` after every API call. Only use `AppLogger.error()` for failures. Use `ToastHelper` for user-facing success feedback.

## Pure Utilities in Helpers, Not Repositories

Date math, formatting, validation utilities should be static methods in helper classes (e.g., `CalendarHelper`, `DateTimeHelper`), not repository methods. Repositories are strictly for data access (API calls, cache reads). If you need a utility function, add it to an existing helper class or create a new one in `utils/helpers/`.

## DateTimeHelper

```dart
import 'package:{pkg}/utils/helpers/datetime_helper.dart';

// Combine date + time controllers to ISO string
final isoString = DateTimeHelper.combineToIsoString(
  dateController: dateCtrl,    // "yyyy-MM-dd"
  timeController: timeCtrl,    // "hh:mm a"
);

// Set controllers from ISO string
DateTimeHelper.setFromIsoString(
  isoString: '2025-06-15T14:00:00.000Z',
  dateController: dateCtrl,
  timeController: timeCtrl,
);

// Parse hours
final hour = DateTimeHelper.parseHourFromTimeString('05:23 PM'); // 17
```

## String Extensions

```dart
import 'package:{pkg}/utils/helpers/string_helper.dart';

'hello world'.toLetterCase        // 'Hello world'
'hello world'.toTitleCase         // 'Hello World'
'item'.sPluralise(2)              // 'items'
['a', 'b', 'c'].toBulletedString  // '• a\n\n• b\n\n• c'
'path/to/file.svg'.isSvg          // true
```

## Error Code Helper

```dart
import 'package:{pkg}/utils/helpers/error_code_helper.dart';

final message = getCustomErrorMessage(errorCode);
```

## DataState<T>

```dart
import 'package:{pkg}/utils/helpers/data_state.dart';

// Creation
const DataState.initial()
const DataState.loading()
const DataState.pageLoading()    // For pagination loading
DataState.loaded(data: result)
DataState.failure(error: message)

// Checking
state.isInitial / state.isNotInitial
state.isLoading / state.isNotLoading
state.isLoaded / state.isNotLoaded
state.isFailure / state.isNotFailure
state.isPageLoading / state.isNotPageLoading
state.isEmpty / state.isNotEmpty
state.hasError / state.hasNoError
state.isError / state.isNotError  // isEmpty || isFailure || hasError

// Data access
state.data         // T? — the actual data
state.error        // dynamic — error object
state.errorMessage // String? — error converted to string

// Transitions (preserve existing data)
state.toLoading()
state.toLoaded(data: newData)
state.toPageLoading()
state.toFailure(error: message)
```

## RepositoryResponse<T>

```dart
import 'package:{pkg}/utils/helpers/repository_response.dart';

class RepositoryResponse<T> {
  final bool isSuccess;
  final T? data;
  final String? message;
}
```

## Common Typedefs

```dart
typedef JsonMap = Map<String, dynamic>;
```

## FocusHandler

For dismissing keyboard:
```dart
FocusHandler.unfocus(context);
```

## PermissionManager

Static utility — no instantiation needed:
```dart
await PermissionManager.requestCameraAndGalleryPermission(
  onGranted: () => // proceed,
  onDenied: (msg) => // show message,
  onPermanentlyDenied: () => // show settings dialog,
);

await PermissionManager.requestLocationPermission(...);
await PermissionManager.requestNotificationPermission();
await PermissionManager.requestMicrophonePermission(...);
final hasLocation = await PermissionManager.hasLocationPermission();
```
