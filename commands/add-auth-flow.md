---
description: Generate a complete authentication flow — login, register, forgot password, OTP verification screens with cubit, state, repository, and route guards.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "[--social] [--otp] [--email-only]"
---

# Generate Auth Flow

Create a complete authentication flow with all necessary screens and logic.

## Arguments

- `$ARGUMENTS`: Auth options. `--social` adds Google/Apple sign-in buttons. `--otp` adds OTP verification. `--email-only` skips social auth.

If not specified, ask the user what auth methods they need.

## Step 1: Detect Existing Auth

Check if auth feature already exists:
- Search for `auth` or `login` in `lib/features/`
- Check `go_router/routes.dart` for auth routes
- Check `core/endpoints/endpoints.dart` for auth endpoints

## Step 2: Create Auth Feature

Generate `lib/features/shared/auth/` with full structure.

### State
```dart
class AuthState extends Equatable {
  const AuthState({
    this.loginData = const DataState.initial(),
    this.registerData = const DataState.initial(),
    this.forgotPasswordData = const DataState.initial(),
    this.otpVerifyData = const DataState.initial(),
    this.isPasswordVisible = false,
    this.isConfirmPasswordVisible = false,
  });

  final DataState<AuthResponse> loginData;
  final DataState<void> registerData;
  final DataState<void> forgotPasswordData;
  final DataState<AuthResponse> otpVerifyData;
  final bool isPasswordVisible;
  final bool isConfirmPasswordVisible;

  // copyWith, props...
}
```

### Cubit Methods
```dart
Future<void> login({required String email, required String password});
Future<void> register({required String name, required String email, required String password});
Future<void> forgotPassword({required String email});
Future<void> verifyOtp({required String email, required String otp});
Future<void> socialLogin({required String provider, required String token});
void togglePasswordVisibility();
Future<void> logout();
```

### Repository Methods
```dart
abstract class AuthRepository {
  Future<RepositoryResponse<AuthResponse>> login({...});
  Future<RepositoryResponse<void>> register({...});
  Future<RepositoryResponse<void>> forgotPassword({...});
  Future<RepositoryResponse<AuthResponse>> verifyOtp({...});
  Future<RepositoryResponse<AuthResponse>> socialLogin({...});
  Future<RepositoryResponse<void>> logout();
}
```

### Screens
1. `auth_login_screen.dart` — Email + password + social buttons
2. `auth_register_screen.dart` — Name + email + password + confirm
3. `auth_forgot_password_screen.dart` — Email input
4. `auth_otp_screen.dart` — PIN input for OTP (if --otp)

### Route Guard

```dart
GoRouter(
  redirect: (context, state) {
    final isLoggedIn = prefs.getToken() != null;
    final isAuthRoute = state.matchedLocation.startsWith('/auth');

    if (!isLoggedIn && !isAuthRoute) return AppRoutes.login;
    if (isLoggedIn && isAuthRoute) return AppRoutes.home;
    return null;
  },
  // ...
)
```

### Token Storage

On successful login/register:
```dart
await prefs.setToken(response.accessToken);
await prefs.setRefreshToken(response.refreshToken);
await prefs.setUserId(response.userId);
await prefs.setUserRole(response.role);
```

On logout:
```dart
prefs.clearAuthData();
context.go(AppRoutes.login);
```

## Step 3: Wire Everything

1. Register cubit in `app_page.dart`
2. Add all auth routes
3. Add auth endpoints
4. Add route guard redirect

## Step 4: Verify

Run `dart analyze lib/` and fix any issues.
