---
name: websocket-patterns
description: WebSocket patterns for codeable_cli projects — SocketService singleton, auto-reconnect, room management, auth handshake, broadcast streams, typed event handling. Use when working with real-time features, websockets, socket connections, or live updates.
---

# WebSocket & SocketService Patterns

You are an expert in the WebSocket patterns used by codeable_cli projects.

## Architecture Overview

```
lib/core/socket_service/
├── socket_service.dart   # Connection manager (singleton via DI)
└── socket_status.dart    # Connection state enum
```

The `SocketService` is a centralized WebSocket connection manager that handles:
- Token-based authentication on connect
- Room join/leave events
- Automatic reconnection with backoff
- Typed broadcast streams for consumers
- Clean disposal lifecycle

## SocketService (DI Singleton)

**CRITICAL**: SocketService is registered as a singleton in `AppModule._setupSocketService()`. Always resolve via DI:

```dart
final SocketService _socketService = Injector.resolve<SocketService>();
```

**Never** instantiate directly — it requires `AppPreferences` for auth tokens.

## Connection Lifecycle

```dart
// Connect (optionally join a room)
await _socketService.connect('room-id');

// Listen to messages
_socketService.messages.listen((Map<String, dynamic> message) {
  final event = message['event'] as String?;
  final data = message['data'];
  // Handle event...
});

// Disconnect (leaves room, cancels listeners)
_socketService.disconnect();

// Refresh connection (disconnect + reconnect with fresh token)
_socketService.refresh();

// Full teardown on logout (disconnect + clear room state)
_socketService.reset();
```

## SocketStatus Enum

```dart
enum SocketStatus {
  disconnected,  // No active connection
  connecting,    // Handshake in progress
  connected,     // Ready to send/receive
  reconnecting,  // Lost connection, auto-retrying
  error,         // Connection failed
}
```

Check connection state:
```dart
if (_socketService.isConnected) { ... }
// or
if (_socketService.status == SocketStatus.connected) { ... }
```

## Configuration

Socket URL is configured **per-flavor** via `AppEnv`, which uses envied-generated classes with per-flavor `.env` files:

```dart
// In each env/.env.{flavor} file:
// SOCKET_URL=wss://api.example.com/ws

// Access via AppEnv:
final socketUrl = AppEnv.socketUrl;
```

**Update `SOCKET_URL`** in each `env/.env.{flavor}` file before using WebSocket features.

## Auth Handshake

On connect, SocketService automatically:
1. Reads auth token from `AppPreferences.getAuthToken()`
2. Sends `{'event': 'auth', 'data': token}` to the server
3. Joins the specified room if a roomId was provided

If no token is available, connect is silently skipped with a warning log.

## Sending Custom Events

```dart
_socketService.sendEvent('chat:message', {
  'roomId': roomId,
  'text': messageText,
  'timestamp': DateTime.now().toIso8601String(),
});
```

This sends `jsonEncode({'event': eventName, 'data': payload})` over the wire.

## Auto-Reconnect

When the connection drops unexpectedly:
1. Status changes to `SocketStatus.reconnecting`
2. A 3-second timer fires, then re-calls `connect()` with the last roomId
3. If disposed, reconnect is suppressed

The reconnect timer is cancelled on explicit `disconnect()` or `dispose()`.

## Pattern: Typed Stream in a Feature

To consume socket events in a specific feature, create a typed stream:

```dart
// In your feature's repository or cubit
class ChatRepository {
  ChatRepository({required SocketService socketService})
    : _socketService = socketService;

  final SocketService _socketService;

  Stream<ChatMessage> get chatMessages => _socketService.messages
      .where((msg) => msg['event'] == 'chat:message')
      .map((msg) => ChatMessage.fromJson(msg['data'] as Map<String, dynamic>));
}
```

## Pattern: Dashboard Real-Time Updates

```dart
class DashboardCubit extends Cubit<DashboardState> {
  DashboardCubit({required SocketService socketService})
    : _socketService = socketService,
      super(const DashboardState()) {
    _subscription = _socketService.messages
        .where((msg) => msg['event'] == 'dashboard:update')
        .listen((msg) {
      final update = DashboardUpdate.fromJson(
        msg['data'] as Map<String, dynamic>,
      );
      emit(state.copyWith(/* apply update */));
    });
  }

  final SocketService _socketService;
  StreamSubscription<Map<String, dynamic>>? _subscription;

  @override
  Future<void> close() {
    _subscription?.cancel();
    return super.close();
  }
}
```

## Pattern: Connect on Login, Reset on Logout

```dart
// After successful login
final socketService = Injector.resolve<SocketService>();
await socketService.connect(userRoomId);

// On logout
socketService.reset(); // Disconnects AND clears room state
```

## Logging

All socket events use `AppLogger` with pretty-printed output:
- `AppLogger.info('SocketService: Connected')`
- `AppLogger.warning('SocketService: Connection lost — reconnecting')`
- `AppLogger.error('SocketService: Connection error', e)`
- `AppLogger.debug('SocketService: Joined room $roomId')`

## Dependencies

- `web_socket_channel: ^3.0.3` (already in pubspec)
- `AppPreferences` for auth token access
- `AppEnv` for socket URL per flavor (envied-based)

## Checklist for WebSocket Integration

1. Update `SOCKET_URL` in each `env/.env.{flavor}` file
2. Ensure backend sends JSON with `{'event': '...', 'data': ...}` format
3. Call `socketService.connect(roomId)` after authentication
4. Subscribe to `socketService.messages` stream in your cubit/repository
5. Filter by event name: `.where((msg) => msg['event'] == 'your:event')`
6. Call `socketService.reset()` on logout
7. Cancel stream subscriptions in cubit `close()` method
