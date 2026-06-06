# Nexa CRM — Arquitectura y Estructura de Carpetas

## Principios Arquitectónicos

- **Clean Architecture** con separación estricta de capas
- **Feature-First Structure** (cada módulo es autocontenido)
- **Repository Pattern** para abstracción de datos
- **Riverpod** para state management reactivo
- **Single Source of Truth** en Firestore

---

## Estructura de Carpetas Completa

```
nexa_crm/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── firebase_options.dart
│   │
│   ├── core/
│   │   ├── constants/
│   │   │   ├── app_colors.dart
│   │   │   ├── app_strings.dart
│   │   │   ├── app_sizes.dart
│   │   │   └── firestore_collections.dart
│   │   ├── errors/
│   │   │   ├── app_exception.dart
│   │   │   └── failure.dart
│   │   ├── extensions/
│   │   │   ├── string_ext.dart
│   │   │   ├── datetime_ext.dart
│   │   │   └── context_ext.dart
│   │   ├── theme/
│   │   │   ├── app_theme.dart
│   │   │   ├── dark_theme.dart
│   │   │   └── text_styles.dart
│   │   ├── router/
│   │   │   ├── app_router.dart
│   │   │   └── app_routes.dart
│   │   ├── utils/
│   │   │   ├── validators.dart
│   │   │   ├── formatters.dart
│   │   │   └── logger.dart
│   │   └── widgets/
│   │       ├── app_button.dart
│   │       ├── app_text_field.dart
│   │       ├── app_card.dart
│   │       ├── app_loader.dart
│   │       ├── empty_state.dart
│   │       ├── error_widget.dart
│   │       ├── kpi_card.dart
│   │       └── status_badge.dart
│   │
│   ├── features/
│   │   │
│   │   ├── auth/
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── auth_remote_datasource.dart
│   │   │   │   ├── models/
│   │   │   │   │   └── user_model.dart
│   │   │   │   └── repositories/
│   │   │   │       └── auth_repository_impl.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── app_user.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── auth_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       ├── login_usecase.dart
│   │   │   │       ├── register_usecase.dart
│   │   │   │       ├── logout_usecase.dart
│   │   │   │       └── reset_password_usecase.dart
│   │   │   └── presentation/
│   │   │       ├── providers/
│   │   │       │   └── auth_provider.dart
│   │   │       ├── screens/
│   │   │       │   ├── login_screen.dart
│   │   │       │   ├── register_screen.dart
│   │   │       │   └── forgot_password_screen.dart
│   │   │       └── widgets/
│   │   │           └── auth_form_wrapper.dart
│   │   │
│   │   ├── dashboard/
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── dashboard_remote_datasource.dart
│   │   │   │   └── repositories/
│   │   │   │       └── dashboard_repository_impl.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── dashboard_stats.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── dashboard_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       └── get_dashboard_stats_usecase.dart
│   │   │   └── presentation/
│   │   │       ├── providers/
│   │   │       │   └── dashboard_provider.dart
│   │   │       ├── screens/
│   │   │       │   └── dashboard_screen.dart
│   │   │       └── widgets/
│   │   │           ├── kpi_section.dart
│   │   │           ├── sales_chart.dart
│   │   │           ├── conversion_chart.dart
│   │   │           ├── activity_feed.dart
│   │   │           └── pipeline_preview.dart
│   │   │
│   │   ├── clients/
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── client_remote_datasource.dart
│   │   │   │   ├── models/
│   │   │   │   │   └── client_model.dart
│   │   │   │   └── repositories/
│   │   │   │       └── client_repository_impl.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── client.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── client_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       ├── get_clients_usecase.dart
│   │   │   │       ├── get_client_by_id_usecase.dart
│   │   │   │       ├── create_client_usecase.dart
│   │   │   │       ├── update_client_usecase.dart
│   │   │   │       └── delete_client_usecase.dart
│   │   │   └── presentation/
│   │   │       ├── providers/
│   │   │       │   └── client_provider.dart
│   │   │       ├── screens/
│   │   │       │   ├── clients_screen.dart
│   │   │       │   ├── client_detail_screen.dart
│   │   │       │   └── client_form_screen.dart
│   │   │       └── widgets/
│   │   │           ├── client_list_tile.dart
│   │   │           ├── client_filter_bar.dart
│   │   │           └── client_avatar.dart
│   │   │
│   │   ├── leads/
│   │   │   └── ... (misma estructura)
│   │   │
│   │   ├── pipeline/
│   │   │   └── ... (misma estructura)
│   │   │
│   │   ├── tasks/
│   │   │   └── ... (misma estructura)
│   │   │
│   │   └── reports/
│   │       └── ... (misma estructura)
│   │
│   └── shared/
│       ├── layout/
│       │   ├── app_shell.dart
│       │   ├── sidebar.dart
│       │   ├── top_bar.dart
│       │   └── responsive_layout.dart
│       └── providers/
│           ├── theme_provider.dart
│           └── navigation_provider.dart
│
├── test/
│   ├── unit/
│   │   ├── auth/
│   │   ├── clients/
│   │   └── dashboard/
│   └── widget/
│
├── pubspec.yaml
├── firebase.json
└── firestore.rules
```
