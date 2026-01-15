---
name: convex-flutter
description: |
  Flutter client for Convex backend with real-time subscriptions, queries, mutations, and actions.
  Use this skill when working with Flutter/Dart apps that need to communicate with a Convex backend.
  This skill covers the convex_flutter package (v3.0.0).
  Key capabilities:
  - Execute queries, mutations, and actions against Convex functions
  - Real-time subscriptions with automatic updates via WebSocket
  - JWT authentication with automatic token refresh
  - WebSocket connection state monitoring
  - App lifecycle handling (pause/resume/reconnect)
  - Cross-platform support (iOS, Android, Web, macOS, Windows, Linux)
---

# Convex Flutter Client

Flutter client for Convex backend. All API details come from `src/convex_flutter/`. Backend guidelines from `convex_rules.txt`.

## Quick Start

```dart
import 'package:convex_flutter/convex_flutter.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: ConvexClient.initialize(
        ConvexConfig(
          deploymentUrl: 'https://your-app.convex.cloud',
          clientId: 'flutter-app-1.0',
        ),
      ),
      builder: (context, snapshot) {
        if (snapshot.hasError) {
          return ErrorWidget(snapshot.error!);
        }
        return const MaterialApp(home: HomeScreen());
      },
    );
  }
}
```

## Configuration

```dart
ConvexConfig(
  deploymentUrl: 'https://your-app.convex.cloud',  // Required
  clientId: 'flutter-app',                          // Optional, defaults to 'flutter-client'
  operationTimeout: Duration(seconds: 30),          // Optional, default 30s
  healthCheckQuery: 'health:ping',                  // Optional
)
```

## Queries

```dart
// Query returns JSON string
final result = await ConvexClient.instance.query('tasks:list', {'limit': '10'});
final decoded = jsonDecode(result);
```

## Mutations

```dart
// Mutations execute with transactional guarantees
await ConvexClient.instance.mutation(
  name: 'tasks:create',
  args: {'title': 'Buy milk', 'completed': false},
);
```

## Actions

```dart
// Actions for side effects (external API calls, etc)
final result = await ConvexClient.instance.action(
  name: 'openai:chat',
  args: {'prompt': 'Hello'},
);
```

## Real-Time Subscriptions

```dart
SubscriptionHandle? _subscription;

_subscription = await ConvexClient.instance.subscribe(
  name: 'messages:list',
  args: {'channelId': 'general'},
  onUpdate: (value) {
    final data = jsonDecode(value);
    setState(() => _messages = data);
  },
  onError: (message, value) {
    debugPrint('Subscription error: $message');
  },
);

// Cancel when done
_subscription?.cancel();
```

## Authentication

### Static Token

```dart
await ConvexClient.instance.setAuth(token: 'your-jwt-token');
await ConvexClient.instance.setAuth(token: null);  // Clear
```

### Auto-Refresh (Recommended)

```dart
final authHandle = await ConvexClient.instance.setAuthWithRefresh(
  fetchToken: () async {
    return await FirebaseAuth.instance.currentUser?.getIdToken();
  },
  onAuthChange: (isAuthenticated) {
    print('Auth state: $isAuthenticated');
  },
);

authHandle.dispose();  // Sign out
```

### Auth State

```dart
ConvexClient.instance.authState.listen((isAuthenticated) {
  setState(() => _isLoggedIn = isAuthenticated);
});
bool isAuthenticated = ConvexClient.instance.isAuthenticated;
```

## Connection State

```dart
ConvexClient.instance.connectionState.listen((state) {
  if (state == WebSocketConnectionState.connected) {
    print('Connected');
  }
});

WebSocketConnectionState current = ConvexClient.instance.currentConnectionState;
bool isConnected = ConvexClient.instance.isConnected;
```

## App Lifecycle

```dart
ConvexClient.instance.lifecycleEvents.listen((event) {
  if (event == AppLifecycleEvent.resumed) {
    ConvexClient.instance.reconnect();
  }
});
```

## Cleanup

```dart
ConvexClient.instance.dispose();
```

## Error Handling

```dart
try {
  await ConvexClient.instance.query('slowQuery', {});
} on TimeoutException {
  print('Operation timed out');
}
```

## Backend Development

See `convex_rules.txt` for Convex TypeScript guidelines:

- Function syntax: `query`, `mutation`, `action` with validators
- Schema design in `convex/schema.ts`
- Validators: `v.string()`, `v.id()`, `v.array()`, `v.object()`, etc.
- Use `withIndex` instead of `filter` in queries
- Internal functions: `internalQuery`, `internalMutation`, `internalAction`

## Sources

- `src/convex_flutter/` - Flutter client source
- `src/convex-rs/` - Rust SDK (not directly used, for reference)
- `convex_rules.txt` - Convex backend guidelines
