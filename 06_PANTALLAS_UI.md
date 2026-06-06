# ═══════════════════════════════════════════════
# PANTALLAS UI — Autenticación · Dashboard · Clientes
# ═══════════════════════════════════════════════

---
# lib/features/auth/presentation/screens/login_screen.dart
---

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../../../../core/constants/app_colors.dart';
import '../../../../core/utils/validators.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../providers/auth_provider.dart';

class LoginScreen extends ConsumerStatefulWidget {
  const LoginScreen({super.key});
  @override
  ConsumerState<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends ConsumerState<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailCtrl = TextEditingController();
  final _passwordCtrl = TextEditingController();
  bool _obscurePassword = true;

  @override
  void dispose() {
    _emailCtrl.dispose(); _passwordCtrl.dispose(); super.dispose();
  }

  Future<void> _submit() async {
    if (!_formKey.currentState!.validate()) return;
    final success = await ref.read(authNotifierProvider.notifier).login(
      _emailCtrl.text, _passwordCtrl.text,
    );
    if (success && mounted) context.go('/dashboard');
  }

  @override
  Widget build(BuildContext context) {
    final authState = ref.watch(authNotifierProvider);
    final isLoading = authState is AsyncLoading;

    // Mostrar error si hay
    ref.listen<AsyncValue<AppUser?>>(authNotifierProvider, (_, next) {
      next.whenOrNull(
        error: (err, _) => ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text(err.toString()), backgroundColor: AppColors.error),
        ),
      );
    });

    return Scaffold(
      backgroundColor: AppColors.backgroundDark,
      body: Center(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(24),
          child: ConstrainedBox(
            constraints: const BoxConstraints(maxWidth: 400),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Logo
                Row(children: [
                  Container(
                    width: 44, height: 44,
                    decoration: BoxDecoration(
                      gradient: const LinearGradient(
                        colors: [AppColors.primary, AppColors.secondary],
                        begin: Alignment.topLeft, end: Alignment.bottomRight,
                      ),
                      borderRadius: BorderRadius.circular(10),
                    ),
                    child: const Center(child: Text('N',
                      style: TextStyle(color: Colors.white, fontSize: 22, fontWeight: FontWeight.bold),
                    )),
                  ),
                  const SizedBox(width: 12),
                  RichText(text: const TextSpan(
                    style: TextStyle(fontSize: 22, fontWeight: FontWeight.w600, color: Colors.white),
                    children: [
                      TextSpan(text: 'Nexa', style: TextStyle(color: AppColors.primaryLight)),
                      TextSpan(text: ' CRM'),
                    ],
                  )),
                ]),
                const SizedBox(height: 40),
                const Text('Bienvenido',
                  style: TextStyle(fontSize: 26, fontWeight: FontWeight.w600, color: Colors.white),
                ),
                const SizedBox(height: 6),
                const Text('Ingresá con tu cuenta corporativa',
                  style: TextStyle(fontSize: 14, color: AppColors.textSecondaryDark),
                ),
                const SizedBox(height: 32),
                Form(
                  key: _formKey,
                  child: Column(children: [
                    AppTextField(
                      controller: _emailCtrl,
                      label: 'Email corporativo',
                      hint: 'tu@empresa.com',
                      keyboardType: TextInputType.emailAddress,
                      prefixIcon: Icons.email_outlined,
                      validator: Validators.email,
                    ),
                    const SizedBox(height: 16),
                    AppTextField(
                      controller: _passwordCtrl,
                      label: 'Contraseña',
                      hint: '••••••••',
                      obscureText: _obscurePassword,
                      prefixIcon: Icons.lock_outline,
                      suffixIcon: _obscurePassword
                          ? Icons.visibility_outlined
                          : Icons.visibility_off_outlined,
                      onSuffixTap: () => setState(() => _obscurePassword = !_obscurePassword),
                      validator: (v) => v?.isEmpty == true ? 'Contraseña requerida' : null,
                    ),
                  ]),
                ),
                const SizedBox(height: 12),
                Align(
                  alignment: Alignment.centerRight,
                  child: TextButton(
                    onPressed: () => context.push('/forgot-password'),
                    child: const Text('¿Olvidaste tu contraseña?',
                      style: TextStyle(fontSize: 12, color: AppColors.primaryLight),
                    ),
                  ),
                ),
                const SizedBox(height: 20),
                AppButton(
                  label: 'Ingresar',
                  onPressed: _submit,
                  isLoading: isLoading,
                  fullWidth: true,
                ),
                const SizedBox(height: 20),
                Center(
                  child: TextButton(
                    onPressed: () => context.push('/register'),
                    child: RichText(text: const TextSpan(
                      style: TextStyle(fontSize: 13, color: AppColors.textSecondaryDark),
                      children: [
                        TextSpan(text: '¿No tenés cuenta? '),
                        TextSpan(text: 'Registrarse',
                          style: TextStyle(color: AppColors.primaryLight, fontWeight: FontWeight.w500),
                        ),
                      ],
                    )),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

---
# lib/shared/layout/app_shell.dart
---
# (Layout principal con sidebar + topbar responsive)

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../../core/constants/app_colors.dart';
import '../providers/navigation_provider.dart';
import '../providers/theme_provider.dart';
import 'sidebar.dart';

class AppShell extends ConsumerWidget {
  final Widget child;
  const AppShell({super.key, required this.child});

  static const double _sidebarWidth = 220;
  static const double _breakpoint = 900;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final width = MediaQuery.of(context).size.width;
    final isDesktop = width >= _breakpoint;

    if (isDesktop) {
      return Scaffold(
        body: Row(children: [
          const SizedBox(width: _sidebarWidth, child: AppSidebar()),
          Expanded(child: child),
        ]),
      );
    }
    // Mobile: drawer
    return Scaffold(
      drawer: const Drawer(child: AppSidebar()),
      body: child,
    );
  }
}

---
# lib/shared/layout/sidebar.dart
---

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../../core/constants/app_colors.dart';
import '../../core/constants/app_routes.dart';

class _NavItem {
  final String label;
  final IconData icon;
  final String route;
  final int? badge;
  const _NavItem({required this.label, required this.icon,
    required this.route, this.badge});
}

class AppSidebar extends ConsumerWidget {
  const AppSidebar({super.key});

  static const _mainItems = [
    _NavItem(label: 'Dashboard', icon: Icons.dashboard_outlined, route: '/dashboard'),
    _NavItem(label: 'Clientes', icon: Icons.people_outline, route: '/clientes', badge: 247),
    _NavItem(label: 'Leads', icon: Icons.electric_bolt_outlined, route: '/leads', badge: 18),
    _NavItem(label: 'Pipeline', icon: Icons.view_kanban_outlined, route: '/pipeline'),
  ];
  static const _mgmtItems = [
    _NavItem(label: 'Tareas', icon: Icons.check_circle_outline, route: '/tareas', badge: 5),
    _NavItem(label: 'Reportes', icon: Icons.bar_chart_outlined, route: '/reportes'),
  ];

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final currentRoute = GoRouterState.of(context).uri.path;

    return Container(
      decoration: BoxDecoration(
        color: AppColors.surfaceDark,
        border: Border(right: BorderSide(color: AppColors.borderDark)),
      ),
      child: Column(children: [
        // Logo
        Padding(
          padding: const EdgeInsets.fromLTRB(18, 20, 18, 16),
          child: Row(children: [
            Container(
              width: 32, height: 32,
              decoration: BoxDecoration(
                gradient: const LinearGradient(
                  colors: [AppColors.primary, AppColors.secondary],
                  begin: Alignment.topLeft, end: Alignment.bottomRight,
                ),
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Center(child: Text('N',
                style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold),
              )),
            ),
            const SizedBox(width: 10),
            RichText(text: const TextSpan(
              style: TextStyle(fontSize: 15, fontWeight: FontWeight.w600, color: Colors.white),
              children: [
                TextSpan(text: 'Nexa', style: TextStyle(color: AppColors.primaryLight)),
                TextSpan(text: ' CRM'),
              ],
            )),
          ]),
        ),
        const Divider(color: AppColors.borderDark, height: 1),
        // Nav Items
        Expanded(
          child: Padding(
            padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 12),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                _navSection('Principal'),
                ..._mainItems.map((item) => _navItem(context, item, currentRoute)),
                const SizedBox(height: 8),
                _navSection('Gestión'),
                ..._mgmtItems.map((item) => _navItem(context, item, currentRoute)),
                const Spacer(),
                _navSection('Config'),
                _navItem(context, const _NavItem(
                  label: 'Configuración', icon: Icons.settings_outlined, route: '/config',
                ), currentRoute),
              ],
            ),
          ),
        ),
        // User Card
        Padding(
          padding: const EdgeInsets.all(8),
          child: Container(
            padding: const EdgeInsets.all(10),
            decoration: BoxDecoration(
              color: AppColors.surfaceDark2,
              borderRadius: BorderRadius.circular(10),
              border: Border.all(color: AppColors.borderDark),
            ),
            child: Row(children: [
              Container(
                width: 30, height: 30,
                decoration: BoxDecoration(
                  gradient: const LinearGradient(
                    colors: [Color(0xFF6366F1), Color(0xFF8B5CF6)],
                    begin: Alignment.topLeft, end: Alignment.bottomRight,
                  ),
                  shape: BoxShape.circle,
                ),
                child: const Center(child: Text('MA',
                  style: TextStyle(color: Colors.white, fontSize: 11, fontWeight: FontWeight.w600),
                )),
              ),
              const SizedBox(width: 8),
              Expanded(child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: const [
                  Text('Matías Admin', style: TextStyle(fontSize: 12, fontWeight: FontWeight.w500, color: Colors.white), overflow: TextOverflow.ellipsis),
                  Text('Administrador', style: TextStyle(fontSize: 10, color: AppColors.textTertiaryDark)),
                ],
              )),
            ]),
          ),
        ),
      ]),
    );
  }

  Widget _navSection(String label) => Padding(
    padding: const EdgeInsets.fromLTRB(10, 10, 0, 4),
    child: Text(label.toUpperCase(),
      style: const TextStyle(fontSize: 10, fontWeight: FontWeight.w500,
          color: AppColors.textTertiaryDark, letterSpacing: .8),
    ),
  );

  Widget _navItem(BuildContext context, _NavItem item, String currentRoute) {
    final isActive = currentRoute.startsWith(item.route);
    return GestureDetector(
      onTap: () => context.go(item.route),
      child: AnimatedContainer(
        duration: const Duration(milliseconds: 150),
        margin: const EdgeInsets.symmetric(vertical: 1),
        decoration: BoxDecoration(
          color: isActive ? AppColors.primaryGlow : Colors.transparent,
          borderRadius: BorderRadius.circular(6),
          border: isActive
              ? Border(left: BorderSide(color: AppColors.primaryLight, width: 3))
              : null,
        ),
        padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 8),
        child: Row(children: [
          Icon(item.icon, size: 16,
            color: isActive ? AppColors.primaryLight : AppColors.textSecondaryDark,
          ),
          const SizedBox(width: 10),
          Expanded(child: Text(item.label,
            style: TextStyle(
              fontSize: 12.5,
              fontWeight: isActive ? FontWeight.w500 : FontWeight.w400,
              color: isActive ? AppColors.primaryLight : AppColors.textSecondaryDark,
            ),
          )),
          if (item.badge != null)
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
              decoration: BoxDecoration(
                color: isActive ? AppColors.primary : AppColors.surfaceDark3,
                borderRadius: BorderRadius.circular(20),
              ),
              child: Text('${item.badge}',
                style: TextStyle(fontSize: 10, fontWeight: FontWeight.w600,
                  color: isActive ? Colors.white : AppColors.textTertiaryDark),
              ),
            ),
        ]),
      ),
    );
  }
}

---
# lib/features/clients/presentation/screens/clients_screen.dart
---

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../../core/constants/app_colors.dart';
import '../../../../core/widgets/app_button.dart';
import '../../domain/entities/client.dart';
import '../providers/client_provider.dart';
import '../widgets/client_list_tile.dart';
import '../widgets/client_filter_bar.dart';
import 'client_form_screen.dart';

class ClientsScreen extends ConsumerWidget {
  const ClientsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final clientsAsync = ref.watch(clientsStreamProvider);
    final filter = ref.watch(clientFilterProvider);

    return Column(children: [
      // Top bar
      Container(
        height: 54,
        padding: const EdgeInsets.symmetric(horizontal: 20),
        decoration: BoxDecoration(
          color: AppColors.surfaceDark,
          border: Border(bottom: BorderSide(color: AppColors.borderDark)),
        ),
        child: Row(children: [
          Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: const [
              Text('Clientes', style: TextStyle(fontSize: 15, fontWeight: FontWeight.w600)),
              Text('247 clientes registrados', style: TextStyle(fontSize: 11, color: AppColors.textSecondaryDark)),
            ],
          ),
          const Spacer(),
          // Search
          SizedBox(
            width: 220,
            child: TextField(
              onChanged: (v) => ref.read(clientFilterProvider.notifier).setSearch(v),
              style: const TextStyle(fontSize: 12),
              decoration: InputDecoration(
                hintText: 'Buscar clientes...',
                prefixIcon: const Icon(Icons.search, size: 16),
                contentPadding: const EdgeInsets.symmetric(vertical: 0, horizontal: 12),
                isDense: true,
              ),
            ),
          ),
          const SizedBox(width: 10),
          OutlinedButton.icon(
            onPressed: () {}, // Export logic
            icon: const Icon(Icons.download_outlined, size: 16),
            label: const Text('Exportar', style: TextStyle(fontSize: 12)),
          ),
          const SizedBox(width: 8),
          ElevatedButton.icon(
            onPressed: () => Navigator.of(context).push(
              MaterialPageRoute(builder: (_) => const ClientFormScreen()),
            ),
            icon: const Icon(Icons.add, size: 16),
            label: const Text('Nuevo Cliente', style: TextStyle(fontSize: 12)),
          ),
        ]),
      ),
      // Filters
      const ClientFilterBar(),
      // Table
      Expanded(
        child: clientsAsync.when(
          loading: () => const Center(child: CircularProgressIndicator()),
          error: (e, _) => Center(child: Text('Error: $e')),
          data: (clients) {
            if (clients.isEmpty) {
              return const Center(child: Text('No hay clientes',
                style: TextStyle(color: AppColors.textSecondaryDark),
              ));
            }
            return SingleChildScrollView(
              padding: const EdgeInsets.all(16),
              child: Container(
                decoration: BoxDecoration(
                  color: AppColors.surfaceDark,
                  border: Border.all(color: AppColors.borderDark),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: Column(
                  children: clients.map((c) => ClientListTile(client: c)).toList(),
                ),
              ),
            );
          },
        ),
      ),
    ]);
  }
}

---
# lib/features/clients/presentation/screens/client_form_screen.dart
---

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:uuid/uuid.dart';
import '../../../../core/constants/app_colors.dart';
import '../../../../core/utils/validators.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../domain/entities/client.dart';
import '../providers/client_provider.dart';

class ClientFormScreen extends ConsumerStatefulWidget {
  final Client? existing; // null = crear, !null = editar
  const ClientFormScreen({super.key, this.existing});

  @override
  ConsumerState<ClientFormScreen> createState() => _ClientFormScreenState();
}

class _ClientFormScreenState extends ConsumerState<ClientFormScreen> {
  final _formKey = GlobalKey<FormState>();
  late final TextEditingController _nombre   = TextEditingController(text: widget.existing?.nombre);
  late final TextEditingController _apellido  = TextEditingController(text: widget.existing?.apellido);
  late final TextEditingController _empresa   = TextEditingController(text: widget.existing?.empresa);
  late final TextEditingController _cuit      = TextEditingController(text: widget.existing?.cuit);
  late final TextEditingController _telefono  = TextEditingController(text: widget.existing?.telefono);
  late final TextEditingController _whatsapp  = TextEditingController(text: widget.existing?.whatsapp);
  late final TextEditingController _email     = TextEditingController(text: widget.existing?.email);
  late final TextEditingController _direccion = TextEditingController(text: widget.existing?.direccion);
  late final TextEditingController _obs       = TextEditingController(text: widget.existing?.observaciones);

  ClientCategory _categoria = ClientCategory.pyme;
  ClientStatus _estado = ClientStatus.activo;

  bool get isEditing => widget.existing != null;

  @override
  void initState() {
    super.initState();
    if (isEditing) {
      _categoria = widget.existing!.categoria;
      _estado = widget.existing!.estado;
    }
  }

  @override
  void dispose() {
    for (final c in [_nombre,_apellido,_empresa,_cuit,_telefono,_whatsapp,_email,_direccion,_obs])
      c.dispose();
    super.dispose();
  }

  Future<void> _submit() async {
    if (!_formKey.currentState!.validate()) return;
    final client = Client(
      id: widget.existing?.id ?? const Uuid().v4(),
      nombre: _nombre.text.trim(),
      apellido: _apellido.text.trim(),
      empresa: _empresa.text.trim().isEmpty ? null : _empresa.text.trim(),
      cuit: _cuit.text.trim().isEmpty ? null : _cuit.text.trim(),
      telefono: _telefono.text.trim().isEmpty ? null : _telefono.text.trim(),
      whatsapp: _whatsapp.text.trim().isEmpty ? null : _whatsapp.text.trim(),
      email: _email.text.trim().isEmpty ? null : _email.text.trim(),
      direccion: _direccion.text.trim().isEmpty ? null : _direccion.text.trim(),
      observaciones: _obs.text.trim().isEmpty ? null : _obs.text.trim(),
      categoria: _categoria,
      estado: _estado,
      fechaAlta: widget.existing?.fechaAlta ?? DateTime.now(),
      createdBy: 'current_user_id', // reemplazar con auth provider
    );

    final notifier = ref.read(clientActionsProvider.notifier);
    final success = isEditing
        ? await notifier.update(client)
        : await notifier.create(client);

    if (success && mounted) {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(
        content: Text(isEditing ? 'Cliente actualizado ✓' : 'Cliente creado ✓'),
        backgroundColor: AppColors.success,
      ));
      Navigator.of(context).pop();
    }
  }

  @override
  Widget build(BuildContext context) {
    final state = ref.watch(clientActionsProvider);
    final isLoading = state is AsyncLoading;

    ref.listen<AsyncValue<void>>(clientActionsProvider, (_, next) {
      next.whenOrNull(error: (err, _) => ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(err.toString()), backgroundColor: AppColors.error),
      ));
    });

    return Scaffold(
      appBar: AppBar(
        title: Text(isEditing ? 'Editar Cliente' : 'Nuevo Cliente'),
        backgroundColor: AppColors.surfaceDark,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(20),
        child: Center(
          child: ConstrainedBox(
            constraints: const BoxConstraints(maxWidth: 640),
            child: Form(
              key: _formKey,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  _section('Información Personal', [
                    _row(AppTextField(controller: _nombre, label: 'Nombre *', hint: 'Juan',
                            validator: (v) => Validators.required(v, 'Nombre')),
                         AppTextField(controller: _apellido, label: 'Apellido *', hint: 'García',
                            validator: (v) => Validators.required(v, 'Apellido'))),
                    const SizedBox(height: 12),
                    AppTextField(controller: _empresa, label: 'Empresa', hint: 'Transportes García S.A.'),
                    const SizedBox(height: 12),
                    AppTextField(controller: _cuit, label: 'CUIT', hint: '20-12345678-9', validator: Validators.cuit),
                  ]),
                  const SizedBox(height: 20),
                  _section('Contacto', [
                    _row(AppTextField(controller: _telefono, label: 'Teléfono', hint: '+54 11 4567-8901',
                            validator: Validators.phone, keyboardType: TextInputType.phone),
                         AppTextField(controller: _whatsapp, label: 'WhatsApp', hint: '+54 9 11 4567-8901',
                            keyboardType: TextInputType.phone)),
                    const SizedBox(height: 12),
                    AppTextField(controller: _email, label: 'Email', hint: 'juan@empresa.com',
                        keyboardType: TextInputType.emailAddress, validator: Validators.email),
                    const SizedBox(height: 12),
                    AppTextField(controller: _direccion, label: 'Dirección', hint: 'Av. Corrientes 1234, CABA'),
                  ]),
                  const SizedBox(height: 20),
                  _section('Clasificación', [
                    _row(
                      _dropdown<ClientCategory>(
                        label: 'Categoría', value: _categoria,
                        items: ClientCategory.values,
                        labelOf: (v) => v.name[0].toUpperCase() + v.name.substring(1),
                        onChanged: (v) => setState(() => _categoria = v!),
                      ),
                      _dropdown<ClientStatus>(
                        label: 'Estado', value: _estado,
                        items: ClientStatus.values,
                        labelOf: (v) => v.name[0].toUpperCase() + v.name.substring(1),
                        onChanged: (v) => setState(() => _estado = v!),
                      ),
                    ),
                  ]),
                  const SizedBox(height: 20),
                  _section('Observaciones', [
                    TextFormField(
                      controller: _obs,
                      maxLines: 3,
                      style: const TextStyle(fontSize: 13),
                      decoration: const InputDecoration(hintText: 'Notas internas sobre el cliente...'),
                    ),
                  ]),
                  const SizedBox(height: 24),
                  Row(mainAxisAlignment: MainAxisAlignment.end, children: [
                    OutlinedButton(
                      onPressed: () => Navigator.pop(context),
                      child: const Text('Cancelar'),
                    ),
                    const SizedBox(width: 12),
                    ElevatedButton(
                      onPressed: isLoading ? null : _submit,
                      child: isLoading
                          ? const SizedBox(width: 16, height: 16,
                              child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white))
                          : Text(isEditing ? 'Actualizar' : 'Guardar Cliente'),
                    ),
                  ]),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }

  Widget _section(String title, List<Widget> children) => Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text(title, style: const TextStyle(fontSize: 13, fontWeight: FontWeight.w600)),
      const SizedBox(height: 12),
      Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: AppColors.surfaceDark,
          border: Border.all(color: AppColors.borderDark),
          borderRadius: BorderRadius.circular(10),
        ),
        child: Column(children: children),
      ),
    ],
  );

  Widget _row(Widget a, Widget b) => Row(children: [
    Expanded(child: a), const SizedBox(width: 12), Expanded(child: b),
  ]);

  Widget _dropdown<T>({
    required String label,
    required T value,
    required List<T> items,
    required String Function(T) labelOf,
    required void Function(T?) onChanged,
  }) => Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text(label, style: const TextStyle(fontSize: 12, fontWeight: FontWeight.w500,
          color: AppColors.textSecondaryDark)),
      const SizedBox(height: 4),
      DropdownButtonFormField<T>(
        value: value,
        onChanged: onChanged,
        items: items.map((e) => DropdownMenuItem(value: e, child: Text(labelOf(e)))).toList(),
        style: const TextStyle(fontSize: 13),
        decoration: const InputDecoration(contentPadding: EdgeInsets.symmetric(horizontal: 12, vertical: 10)),
      ),
    ],
  );
}
