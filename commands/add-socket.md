---
description: Add WebSocket/real-time connection to a feature — creates socket service, wires to cubit for live data updates, handles reconnection and disposal.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> <socket_url>"
---

# Add WebSocket Connection

Add real-time data via WebSocket to a feature.

## Arguments

- `$ARGUMENTS`: Feature path and WebSocket URL.

## Step 1: Create Socket Service

`lib/core/services/socket_service.dart`:

```dart
import 'dart:async';
import 'dart:convert';
import 'package:web_socket_channel/web_socket_channel.dart';

class SocketService {
  WebSocketChannel? _channel;
  final _messageController = StreamController<Map<String, dynamic>>.broadcast();
  Timer? _reconnectTimer;
  bool _isConnected = false;
  String? _url;

  Stream<Map<String, dynamic>> get messages => _messageController.stream;
  bool get isConnected => _isConnected;

  Future<void> connect(String url, {String? token}) async {
    _url = url;
    try {
      final uri = Uri.parse(url);
      _channel = WebSocketChannel.connect(
        uri,
        protocols: token != null ? ['Bearer', token] : null,
      );

      _isConnected = true;
      _channel!.stream.listen(
        (data) {
          final json = jsonDecode(data as String) as Map<String, dynamic>;
          _messageController.add(json);
        },
        onError: (error) {
          _isConnected = false;
          _scheduleReconnect();
        },
        onDone: () {
          _isConnected = false;
          _scheduleReconnect();
        },
      );
    } catch (e) {
      _isConnected = false;
      _scheduleReconnect();
    }
  }

  void send(Map<String, dynamic> data) {
    if (_isConnected && _channel != null) {
      _channel!.sink.add(jsonEncode(data));
    }
  }

  void _scheduleReconnect() {
    _reconnectTimer?.cancel();
    _reconnectTimer = Timer(const Duration(seconds: 5), () {
      if (_url != null) connect(_url!);
    });
  }

  Future<void> disconnect() async {
    _reconnectTimer?.cancel();
    _isConnected = false;
    await _channel?.sink.close();
    _channel = null;
  }

  void dispose() {
    disconnect();
    _messageController.close();
  }
}
```

## Step 2: Wire to Cubit

```dart
class FeatureCubit extends Cubit<FeatureState> {
  FeatureCubit({required this.repository}) : super(const FeatureState());

  final FeatureRepository repository;
  final SocketService _socket = SocketService();
  StreamSubscription? _socketSubscription;

  Future<void> connectSocket() async {
    final token = Injector.resolve<AppPreferences>().getToken();
    await _socket.connect('wss://api.example.com/ws', token: token);

    _socketSubscription = _socket.messages.listen((data) {
      final type = data['type'] as String?;
      switch (type) {
        case 'new_message':
          _handleNewMessage(data);
        case 'status_update':
          _handleStatusUpdate(data);
      }
    });
  }

  void _handleNewMessage(Map<String, dynamic> data) {
    final message = MessageModel.fromJson(data['payload'] as Map<String, dynamic>);
    final messages = [...(state.messagesData.data ?? []), message];
    emit(state.copyWith(messagesData: DataState.loaded(data: messages)));
  }

  void sendMessage(String text) {
    _socket.send({'type': 'message', 'text': text});
  }

  @override
  Future<void> close() {
    _socketSubscription?.cancel();
    _socket.dispose();
    return super.close();
  }
}
```

## Step 3: Add Dependency

```bash
flutter pub add web_socket_channel
```

## Rules

- Always cancel StreamSubscription in cubit's `close()`.
- Always dispose SocketService.
- Implement auto-reconnect with backoff.
- Send auth token on connect.
- Handle connection state in UI (show online/offline indicator).
