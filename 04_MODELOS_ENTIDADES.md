# ═══════════════════════════════════════════════
# MODELOS — Entidades de Dominio + Data Models
# ═══════════════════════════════════════════════

---
# lib/features/auth/domain/entities/app_user.dart
---

import 'package:equatable/equatable.dart';

enum UserRole { admin, supervisor, vendedor }

class AppUser extends Equatable {
  final String id;
  final String email;
  final String name;
  final String? lastName;
  final String? photoUrl;
  final UserRole role;
  final String? organizationId;
  final DateTime createdAt;
  final bool isActive;

  const AppUser({
    required this.id,
    required this.email,
    required this.name,
    this.lastName,
    this.photoUrl,
    required this.role,
    this.organizationId,
    required this.createdAt,
    this.isActive = true,
  });

  String get fullName => lastName != null ? '$name $lastName' : name;
  String get initials {
    final parts = fullName.trim().split(' ');
    if (parts.length >= 2) return '${parts[0][0]}${parts[1][0]}'.toUpperCase();
    return fullName.isNotEmpty ? fullName[0].toUpperCase() : '?';
  }

  @override
  List<Object?> get props => [id, email, name, lastName, role, isActive];
}

---
# lib/features/auth/data/models/user_model.dart
---

import 'package:cloud_firestore/cloud_firestore.dart';
import '../../domain/entities/app_user.dart';

class UserModel extends AppUser {
  const UserModel({
    required super.id,
    required super.email,
    required super.name,
    super.lastName,
    super.photoUrl,
    required super.role,
    super.organizationId,
    required super.createdAt,
    super.isActive,
  });

  factory UserModel.fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return UserModel(
      id: doc.id,
      email: data['email'] as String,
      name: data['name'] as String,
      lastName: data['lastName'] as String?,
      photoUrl: data['photoUrl'] as String?,
      role: UserRole.values.firstWhere(
        (r) => r.name == data['role'],
        orElse: () => UserRole.vendedor,
      ),
      organizationId: data['organizationId'] as String?,
      createdAt: (data['createdAt'] as Timestamp).toDate(),
      isActive: data['isActive'] as bool? ?? true,
    );
  }

  Map<String, dynamic> toFirestore() => {
    'email': email,
    'name': name,
    'lastName': lastName,
    'photoUrl': photoUrl,
    'role': role.name,
    'organizationId': organizationId,
    'createdAt': Timestamp.fromDate(createdAt),
    'isActive': isActive,
  };
}

---
# lib/features/clients/domain/entities/client.dart
---

import 'package:equatable/equatable.dart';

enum ClientStatus { activo, inactivo, prospecto, lead }
enum ClientCategory { pyme, corporativo, emprendedor, distribuidora, otro }

class Client extends Equatable {
  final String id;
  final String nombre;
  final String apellido;
  final String? empresa;
  final String? telefono;
  final String? whatsapp;
  final String? email;
  final String? direccion;
  final String? cuit;
  final ClientCategory categoria;
  final ClientStatus estado;
  final String? observaciones;
  final DateTime fechaAlta;
  final String createdBy;
  final String? assignedTo;
  final DateTime? ultimaActividad;

  const Client({
    required this.id,
    required this.nombre,
    required this.apellido,
    this.empresa,
    this.telefono,
    this.whatsapp,
    this.email,
    this.direccion,
    this.cuit,
    required this.categoria,
    required this.estado,
    this.observaciones,
    required this.fechaAlta,
    required this.createdBy,
    this.assignedTo,
    this.ultimaActividad,
  });

  String get fullName => '$nombre $apellido';
  String get initials => '${nombre.isNotEmpty ? nombre[0] : ""}${apellido.isNotEmpty ? apellido[0] : ""}'.toUpperCase();

  Client copyWith({
    String? nombre, String? apellido, String? empresa,
    String? telefono, String? whatsapp, String? email,
    String? direccion, String? cuit,
    ClientCategory? categoria, ClientStatus? estado,
    String? observaciones, String? assignedTo,
  }) => Client(
    id: id, fechaAlta: fechaAlta, createdBy: createdBy,
    nombre: nombre ?? this.nombre,
    apellido: apellido ?? this.apellido,
    empresa: empresa ?? this.empresa,
    telefono: telefono ?? this.telefono,
    whatsapp: whatsapp ?? this.whatsapp,
    email: email ?? this.email,
    direccion: direccion ?? this.direccion,
    cuit: cuit ?? this.cuit,
    categoria: categoria ?? this.categoria,
    estado: estado ?? this.estado,
    observaciones: observaciones ?? this.observaciones,
    assignedTo: assignedTo ?? this.assignedTo,
  );

  @override
  List<Object?> get props => [id, nombre, apellido, empresa, email, estado];
}

---
# lib/features/clients/data/models/client_model.dart
---

import 'package:cloud_firestore/cloud_firestore.dart';
import '../../domain/entities/client.dart';

class ClientModel extends Client {
  const ClientModel({
    required super.id,
    required super.nombre,
    required super.apellido,
    super.empresa,
    super.telefono,
    super.whatsapp,
    super.email,
    super.direccion,
    super.cuit,
    required super.categoria,
    required super.estado,
    super.observaciones,
    required super.fechaAlta,
    required super.createdBy,
    super.assignedTo,
    super.ultimaActividad,
  });

  factory ClientModel.fromFirestore(DocumentSnapshot doc) {
    final d = doc.data() as Map<String, dynamic>;
    return ClientModel(
      id: doc.id,
      nombre: d['nombre'] as String,
      apellido: d['apellido'] as String,
      empresa: d['empresa'] as String?,
      telefono: d['telefono'] as String?,
      whatsapp: d['whatsapp'] as String?,
      email: d['email'] as String?,
      direccion: d['direccion'] as String?,
      cuit: d['cuit'] as String?,
      categoria: ClientCategory.values.firstWhere(
        (c) => c.name == d['categoria'], orElse: () => ClientCategory.otro,
      ),
      estado: ClientStatus.values.firstWhere(
        (s) => s.name == d['estado'], orElse: () => ClientStatus.prospecto,
      ),
      observaciones: d['observaciones'] as String?,
      fechaAlta: (d['fechaAlta'] as Timestamp).toDate(),
      createdBy: d['createdBy'] as String,
      assignedTo: d['assignedTo'] as String?,
      ultimaActividad: d['ultimaActividad'] != null
          ? (d['ultimaActividad'] as Timestamp).toDate()
          : null,
    );
  }

  factory ClientModel.fromEntity(Client c) => ClientModel(
    id: c.id, nombre: c.nombre, apellido: c.apellido,
    empresa: c.empresa, telefono: c.telefono, whatsapp: c.whatsapp,
    email: c.email, direccion: c.direccion, cuit: c.cuit,
    categoria: c.categoria, estado: c.estado,
    observaciones: c.observaciones, fechaAlta: c.fechaAlta,
    createdBy: c.createdBy, assignedTo: c.assignedTo,
  );

  Map<String, dynamic> toFirestore() => {
    'nombre': nombre,
    'apellido': apellido,
    'empresa': empresa,
    'telefono': telefono,
    'whatsapp': whatsapp,
    'email': email,
    'direccion': direccion,
    'cuit': cuit,
    'categoria': categoria.name,
    'estado': estado.name,
    'observaciones': observaciones,
    'fechaAlta': Timestamp.fromDate(fechaAlta),
    'createdBy': createdBy,
    'assignedTo': assignedTo,
    'ultimaActividad': ultimaActividad != null
        ? Timestamp.fromDate(ultimaActividad!)
        : null,
    'searchTerms': _buildSearchTerms(),
  };

  List<String> _buildSearchTerms() {
    final terms = <String>{};
    void add(String? s) {
      if (s == null) return;
      terms.add(s.toLowerCase());
      final words = s.toLowerCase().split(' ');
      terms.addAll(words);
      for (int i = 1; i <= s.length; i++) {
        terms.add(s.substring(0, i).toLowerCase());
      }
    }
    add(nombre); add(apellido); add(empresa); add(email); add(cuit);
    return terms.toList();
  }
}

---
# lib/features/leads/domain/entities/lead.dart
---

import 'package:equatable/equatable.dart';

enum LeadStatus { nuevo, contactado, propuestaEnviada, negociacion, ganado, perdido }
enum LeadOrigin { web, referido, linkedin, instagram, llamadaFria, feria, email, otro }

class Lead extends Equatable {
  final String id;
  final String nombre;
  final String? empresa;
  final String? telefono;
  final String? email;
  final LeadOrigin origen;
  final String? interes;
  final double? valorEstimado;
  final LeadStatus estado;
  final String createdBy;
  final String? assignedTo;
  final DateTime createdAt;
  final DateTime? updatedAt;
  final String? clientId;   // Si fue convertido

  const Lead({
    required this.id,
    required this.nombre,
    this.empresa,
    this.telefono,
    this.email,
    required this.origen,
    this.interes,
    this.valorEstimado,
    required this.estado,
    required this.createdBy,
    this.assignedTo,
    required this.createdAt,
    this.updatedAt,
    this.clientId,
  });

  bool get isConverted => clientId != null;

  @override
  List<Object?> get props => [id, nombre, empresa, estado, valorEstimado];
}

---
# lib/features/tasks/domain/entities/task.dart
---

import 'package:equatable/equatable.dart';

enum TaskType { llamada, reunion, seguimiento, cobranza, email }
enum TaskStatus { pendiente, enProgreso, completada, cancelada }
enum TaskPriority { alta, media, baja }

class Task extends Equatable {
  final String id;
  final String titulo;
  final String? descripcion;
  final DateTime fecha;
  final TaskPriority prioridad;
  final String responsable;
  final TaskStatus estado;
  final TaskType tipo;
  final String? clientId;
  final String? leadId;
  final String? dealId;
  final bool tieneRecordatorio;
  final DateTime? recordatorioAt;
  final String createdBy;
  final DateTime createdAt;

  const Task({
    required this.id,
    required this.titulo,
    this.descripcion,
    required this.fecha,
    required this.prioridad,
    required this.responsable,
    required this.estado,
    required this.tipo,
    this.clientId,
    this.leadId,
    this.dealId,
    this.tieneRecordatorio = false,
    this.recordatorioAt,
    required this.createdBy,
    required this.createdAt,
  });

  bool get isOverdue => estado == TaskStatus.pendiente && fecha.isBefore(DateTime.now());
  bool get isDueToday {
    final now = DateTime.now();
    return fecha.year == now.year && fecha.month == now.month && fecha.day == now.day;
  }

  @override
  List<Object?> get props => [id, titulo, estado, prioridad, fecha];
}

---
# lib/features/dashboard/domain/entities/dashboard_stats.dart
---

import 'package:equatable/equatable.dart';

class DashboardStats extends Equatable {
  final int totalClientes;
  final int clientesActivos;
  final int leadsNuevos;
  final int negociosAbiertos;
  final double ventasDelMes;
  final double ventasMesAnterior;
  final List<MonthlySales> ventasMensuales;
  final Map<String, int> leadsByStatus;
  final List<RecentActivity> ultimasActividades;
  final int tareasVencidas;
  final int tareasPendientes;

  const DashboardStats({
    required this.totalClientes,
    required this.clientesActivos,
    required this.leadsNuevos,
    required this.negociosAbiertos,
    required this.ventasDelMes,
    required this.ventasMesAnterior,
    required this.ventasMensuales,
    required this.leadsByStatus,
    required this.ultimasActividades,
    required this.tareasVencidas,
    required this.tareasPendientes,
  });

  double get variacionVentas => ventasMesAnterior > 0
      ? ((ventasDelMes - ventasMesAnterior) / ventasMesAnterior) * 100
      : 0;

  @override
  List<Object?> get props => [totalClientes, ventasDelMes];
}

class MonthlySales extends Equatable {
  final String month;
  final double facturado;
  final double cobrado;
  const MonthlySales({required this.month, required this.facturado, required this.cobrado});
  @override
  List<Object?> get props => [month, facturado, cobrado];
}

class RecentActivity extends Equatable {
  final String id;
  final String type;   // 'new_client' | 'lead_won' | 'task_done' | 'deal_moved'
  final String title;
  final String subtitle;
  final DateTime timestamp;
  const RecentActivity({
    required this.id, required this.type,
    required this.title, required this.subtitle, required this.timestamp,
  });
  @override
  List<Object?> get props => [id, type, timestamp];
}
