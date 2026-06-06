# ═══════════════════════════════════════════════
# pubspec.yaml — Nexa CRM
# ═══════════════════════════════════════════════

name: nexa_crm
description: CRM profesional para PyMEs — Nexa CRM
version: 1.0.0+1
publish_to: none

environment:
  sdk: ">=3.0.0 <4.0.0"
  flutter: ">=3.10.0"

dependencies:
  flutter:
    sdk: flutter
  flutter_web_plugins:
    sdk: flutter

  # Firebase
  firebase_core: ^2.24.2
  firebase_auth: ^4.15.3
  cloud_firestore: ^4.13.6
  firebase_storage: ^11.5.6
  firebase_messaging: ^14.7.9

  # State Management
  flutter_riverpod: ^2.4.9
  riverpod_annotation: ^2.3.3

  # Navigation
  go_router: ^13.0.0

  # UI
  google_fonts: ^6.1.0
  fl_chart: ^0.65.0
  flutter_animate: ^4.3.0
  cached_network_image: ^3.3.0
  shimmer: ^3.0.0
  lottie: ^2.7.0

  # Forms & Validation
  reactive_forms: ^16.1.1

  # Utils
  intl: ^0.19.0
  uuid: ^4.2.1
  equatable: ^2.0.5
  dartz: ^0.10.1
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1

  # Export
  pdf: ^3.10.7
  csv: ^6.0.0
  path_provider: ^2.1.1
  share_plus: ^7.2.1

  # Logging
  logger: ^2.0.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
  build_runner: ^2.4.7
  freezed: ^2.4.5
  json_serializable: ^6.7.1
  riverpod_generator: ^2.3.9
  custom_lint: ^0.5.7
  riverpod_lint: ^2.3.7
  mockito: ^5.4.4

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/icons/
    - assets/animations/
  fonts:
    - family: DM_Sans
      fonts:
        - asset: assets/fonts/DM_Sans/DMSans-Regular.ttf
        - asset: assets/fonts/DM_Sans/DMSans-Medium.ttf
          weight: 500
        - asset: assets/fonts/DM_Sans/DMSans-SemiBold.ttf
          weight: 600

---
# ═══════════════════════════════════════════════
# lib/main.dart
# ═══════════════════════════════════════════════

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';
import 'app.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(
    const ProviderScope(
      child: NexaApp(),
    ),
  );
}

---
# ═══════════════════════════════════════════════
# lib/app.dart
# ═══════════════════════════════════════════════

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'core/router/app_router.dart';
import 'core/theme/app_theme.dart';
import 'shared/providers/theme_provider.dart';

class NexaApp extends ConsumerWidget {
  const NexaApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);
    final themeMode = ref.watch(themeModeProvider);

    return MaterialApp.router(
      title: 'Nexa CRM',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      themeMode: themeMode,
      routerConfig: router,
    );
  }
}
