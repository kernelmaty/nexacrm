# ═══════════════════════════════════════════════
# SETUP · ROADMAP · ESCALABILIDAD
# ═══════════════════════════════════════════════

## Cómo levantar el proyecto desde cero

### 1. Prerequisitos

```bash
# Flutter 3.x instalado y configurado
flutter --version  # debe ser >=3.10.0

# Firebase CLI
npm install -g firebase-tools
firebase login

# FlutterFire CLI
dart pub global activate flutterfire_cli
```

### 2. Crear el proyecto Flutter

```bash
flutter create nexa_crm --org com.nexacrm --platforms web,android,ios
cd nexa_crm
```

### 3. Configurar Firebase

```bash
# Crear proyecto en Firebase Console, luego:
flutterfire configure --project=tu-proyecto-firebase

# Esto genera automáticamente:
# - lib/firebase_options.dart
# - android/app/google-services.json
# - ios/Runner/GoogleService-Info.plist
# - web/index.html (con config de firebase)
```

### 4. Instalar dependencias

```bash
# Copiar el pubspec.yaml del doc 02 y ejecutar:
flutter pub get
```

### 5. Configurar Firestore

```bash
# Inicializar Firebase en el proyecto
firebase init firestore

# Copiar las reglas del doc 07 a firestore.rules
# Copiar los índices del doc 07 a firestore.indexes.json
# Deployar reglas e índices:
firebase deploy --only firestore
```

### 6. Ejecutar el proyecto

```bash
# Web (recomendado para desarrollo)
flutter run -d chrome

# Android
flutter run -d android

# iOS
flutter run -d ios
```

---

## Roadmap por Fases

### FASE 1 — MVP Core (Actual)
- [x] Arquitectura limpia (Clean Architecture + Riverpod)
- [x] Firebase Auth (Login, Registro, Reset Password)
- [x] Dashboard con KPIs y gráficos
- [x] CRUD Clientes completo + búsqueda + filtros
- [x] Navegación con GoRouter
- [x] Layout responsive (Web + Mobile)
- [x] Tema claro/oscuro
- [x] Firestore Rules

### FASE 2 — Módulos Comerciales
- [ ] Módulo Leads con historial de cambios
- [ ] Pipeline Kanban con drag & drop
- [ ] Módulo Tareas con recordatorios (FCM)
- [ ] Ficha detallada del cliente (timeline)
- [ ] Convertir Lead → Cliente (un clic)

### FASE 3 — Reportes y Exportación
- [ ] Reportes con fl_chart (ventas, conversión, embudo)
- [ ] Exportar PDF (con pdf package)
- [ ] Exportar Excel/CSV
- [ ] Filtros por fecha, vendedor, categoría
- [ ] Dashboard de métricas de equipo (para supervisores)

### FASE 4 — Notificaciones y Automatizaciones
- [ ] Firebase Cloud Messaging (notificaciones push)
- [ ] Recordatorios de tareas vencidas
- [ ] Alertas de seguimiento
- [ ] Cloud Functions para automatizaciones
- [ ] Webhooks de integración

### FASE 5 — Integraciones IA y Externas
- [ ] Gemini AI para sugerencias de ventas
- [ ] WhatsApp Business API
- [ ] Google Calendar sync
- [ ] Email Marketing (Mailchimp / SendGrid)
- [ ] Facturación electrónica AFIP Argentina
- [ ] Multi-tenant / Multi-organización

---

## Estructura de Datos en Firestore

### Colección `users`
```
users/{uid}
  email: string
  name: string
  lastName: string?
  role: 'admin' | 'supervisor' | 'vendedor'
  organizationId: string?
  createdAt: timestamp
  isActive: boolean
```

### Colección `clients`
```
clients/{clientId}
  nombre: string
  apellido: string
  empresa: string?
  telefono: string?
  whatsapp: string?
  email: string?
  direccion: string?
  cuit: string?
  categoria: 'pyme' | 'corporativo' | 'emprendedor' | 'distribuidora' | 'otro'
  estado: 'activo' | 'inactivo' | 'prospecto' | 'lead'
  observaciones: string?
  fechaAlta: timestamp
  createdBy: string (uid)
  assignedTo: string? (uid)
  ultimaActividad: timestamp?
  searchTerms: string[] (generado automáticamente)
  
  /activities/{actId}
    type: string
    description: string
    createdBy: string
    createdAt: timestamp
```

### Colección `leads`
```
leads/{leadId}
  nombre: string
  empresa: string?
  telefono: string?
  email: string?
  origen: 'web' | 'referido' | 'linkedin' | 'instagram' | 'llamadaFria' | 'feria'
  interes: string?
  valorEstimado: number?
  estado: 'nuevo' | 'contactado' | 'propuestaEnviada' | 'negociacion' | 'ganado' | 'perdido'
  createdBy: string
  assignedTo: string?
  createdAt: timestamp
  updatedAt: timestamp?
  clientId: string? (si fue convertido)
  
  /history/{histId}
    previousStatus: string
    newStatus: string
    changedBy: string
    changedAt: timestamp
    notes: string?
```

### Colección `tasks`
```
tasks/{taskId}
  titulo: string
  descripcion: string?
  fecha: timestamp
  prioridad: 'alta' | 'media' | 'baja'
  responsable: string (uid)
  estado: 'pendiente' | 'enProgreso' | 'completada' | 'cancelada'
  tipo: 'llamada' | 'reunion' | 'seguimiento' | 'cobranza' | 'email'
  clientId: string?
  leadId: string?
  dealId: string?
  tieneRecordatorio: boolean
  recordatorioAt: timestamp?
  createdBy: string
  createdAt: timestamp
  completedAt: timestamp?
```

### Colección `deals`
```
deals/{dealId}
  titulo: string
  clientId: string?
  leadId: string?
  valor: number
  estado: 'nuevo' | 'contactado' | 'cotizacion' | 'negociacion' | 'cierre' | 'ganado' | 'perdido'
  probabilidad: number (0-100)
  responsable: string (uid)
  fechaCierre: timestamp?
  createdBy: string
  createdAt: timestamp
  updatedAt: timestamp
  
  /notes/{noteId}
    content: string
    createdBy: string
    createdAt: timestamp
```

---

## Decisiones de Arquitectura Importantes

### Por qué Clean Architecture
- **Independencia de frameworks**: el dominio no conoce Flutter ni Firebase
- **Testabilidad**: cada capa se puede testear aisladamente
- **Escalabilidad**: agregar nuevas fuentes de datos (REST API, SQL local) sin tocar la lógica de negocio
- **Preparado para IA**: los use cases son el punto natural para inyectar llamadas a Gemini/GPT

### Por qué Riverpod sobre BLoC
- Menos boilerplate, más legible
- `StreamProvider` nativo para Firestore real-time
- `AutoDispose` automático (evita memory leaks)
- `FutureProvider.family` para queries con parámetros
- Mejor integración con go_router

### Búsqueda full-text con Firestore
Firestore no tiene búsqueda full-text nativa. La estrategia implementada es:
1. Al guardar un cliente, generar un array `searchTerms` con todos los prefijos de cada campo
2. Usar `.where('searchTerms', arrayContains: query)` para búsqueda eficiente
3. Para búsquedas más avanzadas en el futuro: integrar Algolia o Typesense vía Cloud Functions

### Soft Delete
Los clientes nunca se borran físicamente. Se marca `estado: 'eliminado'` y `deletedAt`.
Esto preserva el historial de actividades y permite restaurar datos.

### Seguridad multicapa
1. **Firebase Rules**: primera línea, valida permisos por rol en Firestore
2. **Validators en UI**: feedback inmediato al usuario
3. **Use Cases**: lógica de negocio que puede rechazar operaciones
4. **searchTerms sanitization**: los datos se normalizan (lowercase, sin acentos) antes de guardarse

---

## Widgets Reutilizables Clave

```dart
// Uso típico de AppTextField
AppTextField(
  controller: _emailCtrl,
  label: 'Email corporativo',
  hint: 'tu@empresa.com',
  keyboardType: TextInputType.emailAddress,
  prefixIcon: Icons.email_outlined,
  validator: Validators.email,
);

// Uso típico de KPICard
KPICard(
  label: 'CLIENTES ACTIVOS',
  value: '247',
  change: '+12%',
  isPositive: true,
  icon: Icons.people_outline,
  color: AppColors.primary,
);

// Uso típico de StatusBadge
StatusBadge(
  label: 'Activo',
  color: AppColors.success,
);

// Uso típico de AppButton
AppButton(
  label: 'Guardar Cliente',
  onPressed: _submit,
  isLoading: isLoading,
  fullWidth: true,
);
```

---

## Configuración de FCM para Notificaciones

```dart
// lib/core/services/notification_service.dart
class NotificationService {
  final _messaging = FirebaseMessaging.instance;

  Future<void> initialize() async {
    await _messaging.requestPermission(alert: true, badge: true, sound: true);
    
    // Token del dispositivo (guardar en Firestore para el usuario)
    final token = await _messaging.getToken();
    if (token != null) await _saveToken(token);

    // Handlers
    FirebaseMessaging.onMessage.listen(_handleForeground);
    FirebaseMessaging.onMessageOpenedApp.listen(_handleBackground);
  }

  void _handleForeground(RemoteMessage msg) {
    // Mostrar snackbar o dialog
  }

  void _handleBackground(RemoteMessage msg) {
    // Navegar a la pantalla correspondiente
  }

  Future<void> _saveToken(String token) async {
    final uid = FirebaseAuth.instance.currentUser?.uid;
    if (uid == null) return;
    await FirebaseFirestore.instance.collection('users').doc(uid).update({
      'fcmTokens': FieldValue.arrayUnion([token]),
      'lastTokenUpdate': FieldValue.serverTimestamp(),
    });
  }
}
```

---

## Contacto y Próximos Pasos

Para continuar con la **Fase 2** (Leads + Pipeline Kanban + Tareas con FCM), pedí:
> "Continúa con la Fase 2 de Nexa CRM: módulo leads, pipeline Kanban y sistema de tareas con notificaciones"

Para integración con IA (Gemini):
> "Agrega el módulo de IA a Nexa CRM con Gemini para sugerencias de ventas"

Para facturación electrónica argentina:
> "Implementa integración con AFIP para facturación electrónica en Nexa CRM"
