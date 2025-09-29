# EasyCarConnect – Flutter Scaffold (v1)

This canvas contains a ready-to-copy Flutter starter for **EasyCarConnect** with routing, state management (Riverpod), HTTP client (Dio), models (Freezed/JsonSerializable), and a few feature stubs (Auth, Vehicles, Shipments). Follow the steps and paste the files into your repo.

---

## 0) Prerequisites
- Flutter 3.22+
- Dart 3.4+
- `melos` (optional for multi-package)
- iOS/Android SDKs installed

---

## 1) Create the Flutter app
```bash
flutter create easycarconnect
cd easycarconnect
```

### Recommended tooling
```bash
flutter pub add flutter_riverpod go_router dio intl
flutter pub add --dev build_runner freezed json_serializable freezed_annotation flutter_lints
flutter pub add freezed_annotation
```

> If using Firebase later: add `flutterfire_cli` and run `flutterfire configure`.

---

## 2) Project structure
```
easycarconnect/
  lib/
    app.dart
    app_router.dart
    main.dart
    core/
      theme.dart
      env.dart
      http_providers.dart
    features/
      auth/
        data/auth_repository.dart
        presentation/login_page.dart
      vehicles/
        data/vehicle_repository.dart
        domain/vehicle.dart
        presentation/vehicle_list_page.dart
        presentation/vehicle_detail_page.dart
      shipments/
        presentation/shipment_list_page.dart
```

---

## 3) Files

### `pubspec.yaml` (minimal excerpt)
```yaml
name: easycarconnect
publish_to: 'none'
environment:
  sdk: '>=3.4.0 <4.0.0'
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.1
  go_router: ^14.2.0
  dio: ^5.5.0+1
  intl: ^0.19.0
  freezed_annotation: ^2.4.4

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0
  build_runner: ^2.4.11
  freezed: ^2.5.2
  json_serializable: ^6.8.0
flutter:
  uses-material-design: true
```

### `lib/main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'app.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ProviderScope(child: EasyCarConnectApp()));
}
```

### `lib/app.dart`
```dart
import 'package:flutter/material.dart';
import 'app_router.dart';
import 'core/theme.dart';

class EasyCarConnectApp extends StatelessWidget {
  const EasyCarConnectApp({super.key});

  @override
  Widget build(BuildContext context) {
    final router = createRouter();
    return MaterialApp.router(
      title: 'EasyCarConnect',
      theme: buildTheme(),
      routerConfig: router,
    );
  }
}
```

### `lib/app_router.dart`
```dart
import 'package:flutter/widgets.dart';
import 'package:go_router/go_router.dart';
import 'features/vehicles/presentation/vehicle_list_page.dart';
import 'features/vehicles/presentation/vehicle_detail_page.dart';
import 'features/auth/presentation/login_page.dart';

GoRouter createRouter() => GoRouter(
  initialLocation: '/vehicles',
  routes: [
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginPage(),
    ),
    GoRoute(
      path: '/vehicles',
      builder: (context, state) => const VehicleListPage(),
      routes: [
        GoRoute(
          path: ':id',
          builder: (context, state) => VehicleDetailPage(id: state.pathParameters['id']!),
        ),
      ],
    ),
  ],
  errorBuilder: (context, state) => const Directionality(
    textDirection: TextDirection.ltr,
    child: Center(child: Text('Not found')),
  ),
);
```

### `lib/core/theme.dart`
```dart
import 'package:flutter/material.dart';

ThemeData buildTheme() {
  final base = ThemeData.light(useMaterial3: true);
  return base.copyWith(
    colorScheme: base.colorScheme.copyWith(
      primary: Colors.blue,
      secondary: Colors.indigo,
    ),
    appBarTheme: const AppBarTheme(centerTitle: true),
  );
}
```

### `lib/core/env.dart`
```dart
class Env {
  static const apiBaseUrl = String.fromEnvironment(
    'EASYCARCONNECT_API',
    defaultValue: 'https://api.easycarconnect.local',
  );
}
```

### `lib/core/http_providers.dart`
```dart
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'env.dart';

final dioProvider = Provider<Dio>((ref) {
  final dio = Dio(BaseOptions(
    baseUrl: Env.apiBaseUrl,
    connectTimeout: const Duration(seconds: 10),
    receiveTimeout: const Duration(seconds: 20),
    headers: {'Accept': 'application/json'},
  ));
  dio.interceptors.add(LogInterceptor(responseBody: false));
  return dio;
});
```

---

## Auth (stub)

### `lib/features/auth/presentation/login_page.dart`
```dart
import 'package:flutter/material.dart';

class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            const TextField(decoration: InputDecoration(labelText: 'Email')),
            const SizedBox(height: 8),
            const TextField(decoration: InputDecoration(labelText: 'Password'), obscureText: true),
            const SizedBox(height: 16),
            FilledButton(
              onPressed: () {},
              child: const Text('Sign in'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## Vehicles

### `lib/features/vehicles/domain/vehicle.dart`
```dart
import 'package:freezed_annotation/freezed_annotation.dart';
part 'vehicle.freezed.dart';
part 'vehicle.g.dart';

@freezed
class Vehicle with _$Vehicle {
  const factory Vehicle({
    required String id,
    required String make,
    required String model,
    required int year,
    required double price,
    String? imageUrl,
  }) = _Vehicle;

  factory Vehicle.fromJson(Map<String, dynamic> json) => _$VehicleFromJson(json);
}
```

### `lib/features/vehicles/data/vehicle_repository.dart`
```dart
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../core/http_providers.dart';
import '../domain/vehicle.dart';

final vehicleRepositoryProvider = Provider<VehicleRepository>((ref) {
  final dio = ref.watch(dioProvider);
  return VehicleRepository(dio);
});

class VehicleRepository {
  VehicleRepository(this._dio);
  final Dio _dio;

  Future<List<Vehicle>> list() async {
    final res = await _dio.get('/vehicles');
    final data = (res.data as List).cast<Map<String, dynamic>>();
    return data.map(Vehicle.fromJson).toList();
  }

  Future<Vehicle> getById(String id) async {
    final res = await _dio.get('/vehicles/$id');
    return Vehicle.fromJson(res.data as Map<String, dynamic>);
  }
}
```

### `lib/features/vehicles/presentation/vehicle_list_page.dart`
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../data/vehicle_repository.dart';
import '../domain/vehicle.dart';

final vehiclesProvider = FutureProvider<List<Vehicle>>((ref) async {
  return ref.watch(vehicleRepositoryProvider).list();
});

class VehicleListPage extends ConsumerWidget {
  const VehicleListPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final vehicles = ref.watch(vehiclesProvider);
    return Scaffold(
      appBar: AppBar(title: const Text('Vehicles')),
      body: vehicles.when(
        data: (items) => ListView.separated(
          itemCount: items.length,
          separatorBuilder: (_, __) => const Divider(height: 1),
          itemBuilder: (context, i) {
            final v = items[i];
            return ListTile(
              leading: v.imageUrl != null ? Image.network(v.imageUrl!, width: 56, height: 56, fit: BoxFit.cover) : const Icon(Icons.directions_car),
              title: Text('${v.make} ${v.model}'),
              subtitle: Text('${v.year} • €${v.price.toStringAsFixed(0)}'),
              onTap: () => context.go('/vehicles/${v.id}'),
            );
          },
        ),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, st) => Center(child: Text('Error: $e')),
      ),
    );
  }
}
```

### `lib/features/vehicles/presentation/vehicle_detail_page.dart`
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../data/vehicle_repository.dart';
import '../domain/vehicle.dart';

final vehicleProvider = FutureProvider.family<Vehicle, String>((ref, id) async {
  return ref.watch(vehicleRepositoryProvider).getById(id);
});

class VehicleDetailPage extends ConsumerWidget {
  const VehicleDetailPage({super.key, required this.id});
  final String id;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final asyncV = ref.watch(vehicleProvider(id));
    return Scaffold(
      appBar: AppBar(title: const Text('Vehicle')),
      body: asyncV.when(
        data: (v) => ListView(
          padding: const EdgeInsets.all(16),
          children: [
            if (v.imageUrl != null)
              ClipRRect(
                borderRadius: BorderRadius.circular(12),
                child: Image.network(v.imageUrl!),
              ),
            const SizedBox(height: 16),
            Text('${v.make} ${v.model}', style: Theme.of(context).textTheme.headlineSmall),
            Text('${v.year} • €${v.price.toStringAsFixed(0)}'),
          ],
        ),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, st) => Center(child: Text('Error: $e')),
      ),
    );
  }
}
```

---

## Shipments (placeholder)

### `lib/features/shipments/presentation/shipment_list_page.dart`
```dart
import 'package:flutter/material.dart';

class ShipmentListPage extends StatelessWidget {
  const ShipmentListPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Shipments')),
      body: const Center(child: Text('Coming soon…')),
    );
  }
}
```

---

## 4) Build the models
Run code generation after creating the `Vehicle` model:
```bash
flutter pub run build_runner build -d
```

For watch mode during dev:
```bash
flutter pub run build_runner watch -d
```

---

## 5) Environment (API endpoint)
You can override the API base at build time:
```bash
flutter run -d chrome --dart-define=EASYCARCONNECT_API=https://api.yourdomain.com
```

---

## 6) Next steps
- Plug your real backend endpoints into `VehicleRepository`.
- Add authentication flow (OAuth / email-password, etc.).
- Add search & filters for vehicles (price range, make, model, year).
- Implement shipments/transporters booking UI.
- Add i18n (FR/EN) with `intl` and `flutter_localizations`.
- Set up CI (Flutter test/build) and flavors (dev/stage/prod).

---

## 7) Git init (if starting fresh)
```bash
git init
git add .
git commit -m "feat: bootstrap EasyCarConnect Flutter app"
```

If you already have a repo, replace the `lib/` and `pubspec.yaml` with the above, then commit.

