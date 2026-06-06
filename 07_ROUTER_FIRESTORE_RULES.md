# ═══════════════════════════════════════════════
# ROUTER · FIRESTORE RULES · FIREBASE CONFIG
# ═══════════════════════════════════════════════

---
# lib/core/router/app_router.dart
---

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../../features/auth/presentation/screens/login_screen.dart';
import '../../features/auth/presentation/screens/register_screen.dart';
import '../../features/auth/presentation/screens/forgot_password_screen.dart';
import '../../features/dashboard/presentation/screens/dashboard_screen.dart';
import '../../features/clients/presentation/screens/clients_screen.dart';
import '../../features/clients/presentation/screens/client_detail_screen.dart';
import '../../features/leads/presentation/screens/leads_screen.dart';
import '../../features/pipeline/presentation/screens/pipeline_screen.dart';
import '../../features/tasks/presentation/screens/tasks_screen.dart';
import '../../features/reports/presentation/screens/reports_screen.dart';
import '../../shared/layout/app_shell.dart';
import '../auth/auth_provider.dart';

final appRouterProvider = Provider<GoRouter>((ref) {
  final authState = ref.watch(authStateProvider);

  return GoRouter(
    initialLocation: '/dashboard',
    debugLogDiagnostics: false,
    redirect: (context, state) {
      final isLoggedIn = authState.value != null;
      final isAuthRoute = ['/login', '/register', '/forgot-password']
          .any((r) => state.uri.path.startsWith(r));

      if (!isLoggedIn && !isAuthRoute) return '/login';
      if (isLoggedIn && isAuthRoute) return '/dashboard';
      return null;
    },
    routes: [
      // Auth routes (sin shell)
      GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
      GoRoute(path: '/register', builder: (_, __) => const RegisterScreen()),
      GoRoute(path: '/forgot-password', builder: (_, __) => const ForgotPasswordScreen()),

      // Protected routes (con shell)
      ShellRoute(
        builder: (context, state, child) => AppShell(child: child),
        routes: [
          GoRoute(
            path: '/dashboard',
            builder: (_, __) => const DashboardScreen(),
          ),
          GoRoute(
            path: '/clientes',
            builder: (_, __) => const ClientsScreen(),
            routes: [
              GoRoute(
                path: ':id',
                builder: (_, state) => ClientDetailScreen(
                  clientId: state.pathParameters['id']!,
                ),
              ),
            ],
          ),
          GoRoute(path: '/leads', builder: (_, __) => const LeadsScreen()),
          GoRoute(path: '/pipeline', builder: (_, __) => const PipelineScreen()),
          GoRoute(path: '/tareas', builder: (_, __) => const TasksScreen()),
          GoRoute(path: '/reportes', builder: (_, __) => const ReportsScreen()),
        ],
      ),
    ],
    errorBuilder: (context, state) => Scaffold(
      body: Center(
        child: Text('Página no encontrada: ${state.uri.path}'),
      ),
    ),
  );
});

---
# firestore.rules — Reglas de Seguridad Firestore
---

rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ══════════════════════════════════════════
    // Helper functions
    // ══════════════════════════════════════════

    function isAuth() {
      return request.auth != null;
    }

    function isUser(userId) {
      return request.auth.uid == userId;
    }

    function getUserData() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data;
    }

    function getRole() {
      return getUserData().role;
    }

    function isAdmin() {
      return isAuth() && getRole() == 'admin';
    }

    function isSupervisor() {
      return isAuth() && (getRole() == 'admin' || getRole() == 'supervisor');
    }

    function isVendedor() {
      return isAuth() && getRole() in ['admin', 'supervisor', 'vendedor'];
    }

    function isOwner(resource) {
      return resource.data.createdBy == request.auth.uid;
    }

    function isAssigned(resource) {
      return resource.data.assignedTo == request.auth.uid;
    }

    function isOwnerOrSupervisor(resource) {
      return isOwner(resource) || isSupervisor();
    }

    // Valida que el documento no tenga campos extra no permitidos
    function hasOnlyAllowedFields(allowedFields) {
      return request.resource.data.keys().hasOnly(allowedFields);
    }

    // ══════════════════════════════════════════
    // USERS
    // ══════════════════════════════════════════
    match /users/{userId} {
      allow read: if isAuth();
      allow create: if isAuth() && isUser(userId);
      allow update: if isAdmin() || isUser(userId);
      allow delete: if isAdmin();
    }

    // ══════════════════════════════════════════
    // CLIENTS
    // ══════════════════════════════════════════
    match /clients/{clientId} {
      allow read: if isVendedor();
      allow create: if isVendedor()
        && request.resource.data.nombre is string
        && request.resource.data.apellido is string
        && request.resource.data.createdBy == request.auth.uid
        && request.resource.data.fechaAlta is timestamp;
      allow update: if isOwnerOrSupervisor(resource)
        && request.resource.data.createdBy == resource.data.createdBy;
      allow delete: if isAdmin();

      // Actividades del cliente
      match /activities/{actId} {
        allow read: if isVendedor();
        allow create: if isVendedor();
        allow update, delete: if isAdmin();
      }
    }

    // ══════════════════════════════════════════
    // LEADS
    // ══════════════════════════════════════════
    match /leads/{leadId} {
      allow read: if isVendedor();
      allow create: if isVendedor()
        && request.resource.data.nombre is string
        && request.resource.data.createdBy == request.auth.uid;
      allow update: if isOwnerOrSupervisor(resource);
      allow delete: if isSupervisor();

      match /history/{histId} {
        allow read: if isVendedor();
        allow create: if isVendedor();
        allow update, delete: if isAdmin();
      }
    }

    // ══════════════════════════════════════════
    // TASKS
    // ══════════════════════════════════════════
    match /tasks/{taskId} {
      allow read: if isVendedor();
      allow create: if isVendedor()
        && request.resource.data.titulo is string
        && request.resource.data.createdBy == request.auth.uid;
      allow update: if isOwnerOrSupervisor(resource)
        || isAssigned(resource);
      allow delete: if isOwnerOrSupervisor(resource);
    }

    // ══════════════════════════════════════════
    // DEALS (Pipeline)
    // ══════════════════════════════════════════
    match /deals/{dealId} {
      allow read: if isVendedor();
      allow create: if isVendedor()
        && request.resource.data.createdBy == request.auth.uid;
      allow update: if isOwnerOrSupervisor(resource);
      allow delete: if isSupervisor();

      match /notes/{noteId} {
        allow read: if isVendedor();
        allow create: if isVendedor();
        allow update, delete: if isOwnerOrSupervisor(resource);
      }
    }

    // ══════════════════════════════════════════
    // ACTIVITIES (Log global)
    // ══════════════════════════════════════════
    match /activities/{actId} {
      allow read: if isVendedor();
      allow create: if isVendedor()
        && request.resource.data.createdBy == request.auth.uid;
      allow update, delete: if isAdmin();
    }

    // ══════════════════════════════════════════
    // REPORTS (solo lectura para vendedores)
    // ══════════════════════════════════════════
    match /reports/{reportId} {
      allow read: if isVendedor();
      allow write: if isAdmin();
    }

    // Denegar todo lo demás por defecto
    match /{document=**} {
      allow read, write: if false;
    }
  }
}

---
# Índices de Firestore recomendados (firestore.indexes.json)
---

{
  "indexes": [
    {
      "collectionGroup": "clients",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "estado", "order": "ASCENDING" },
        { "fieldPath": "fechaAlta", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "clients",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "categoria", "order": "ASCENDING" },
        { "fieldPath": "fechaAlta", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "clients",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "searchTerms", "arrayConfig": "CONTAINS" },
        { "fieldPath": "fechaAlta", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "leads",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "estado", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "tasks",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "responsable", "order": "ASCENDING" },
        { "fieldPath": "estado", "order": "ASCENDING" },
        { "fieldPath": "fecha", "order": "ASCENDING" }
      ]
    },
    {
      "collectionGroup": "deals",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "estado", "order": "ASCENDING" },
        { "fieldPath": "updatedAt", "order": "DESCENDING" }
      ]
    }
  ]
}
