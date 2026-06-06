# ═══════════════════════════════════════════════
# REPOSITORIES, SERVICES Y PROVIDERS
# ═══════════════════════════════════════════════

---
# lib/features/clients/domain/repositories/client_repository.dart
---

import 'package:dartz/dartz.dart';
import '../../../../core/errors/failure.dart';
import '../entities/client.dart';

abstract class ClientRepository {
  /// Escucha en tiempo real la lista de clientes
  Stream<Either<Failure, List<Client>>> watchClients({
    ClientStatus? filterStatus,
    ClientCategory? filterCategory,
    String? searchQuery,
    int limit = 50,
  });

  Future<Either<Failure, Client>> getClientById(String id);
  Future<Either<Failure, Client>> createClient(Client client);
  Future<Either<Failure, Client>> updateClient(Client client);
  Future<Either<Failure, void>> deleteClient(String id);
  Future<Either<Failure, List<Client>>> searchClients(String query);
  Future<Either<Failure, String>> exportToCSV(List<Client> clients);
}

---
# lib/features/clients/data/repositories/client_repository_impl.dart
---

import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:dartz/dartz.dart';
import '../../../../core/constants/firestore_collections.dart';
import '../../../../core/errors/failure.dart';
import '../../domain/entities/client.dart';
import '../../domain/repositories/client_repository.dart';
import '../datasources/client_remote_datasource.dart';

class ClientRepositoryImpl implements ClientRepository {
  final ClientRemoteDatasource _datasource;

  ClientRepositoryImpl(this._datasource);

  @override
  Stream<Either<Failure, List<Client>>> watchClients({
    ClientStatus? filterStatus,
    ClientCategory? filterCategory,
    String? searchQuery,
    int limit = 50,
  }) async* {
    try {
      yield* _datasource
          .watchClients(
            filterStatus: filterStatus,
            filterCategory: filterCategory,
            searchQuery: searchQuery,
            limit: limit,
          )
          .map((clients) => Right<Failure, List<Client>>(clients));
    } on FirebaseException catch (e) {
      yield Left(ServerFailure(e.message ?? 'Error de Firebase'));
    } catch (e) {
      yield Left(ServerFailure(e.toString()));
    }
  }

  @override
  Future<Either<Failure, Client>> getClientById(String id) async {
    try {
      final client = await _datasource.getClientById(id);
      if (client == null) return Left(NotFoundFailure('Cliente no encontrado'));
      return Right(client);
    } on FirebaseException catch (e) {
      return Left(ServerFailure(e.message ?? 'Error de Firebase'));
    }
  }

  @override
  Future<Either<Failure, Client>> createClient(Client client) async {
    try {
      final created = await _datasource.createClient(client);
      return Right(created);
    } on FirebaseException catch (e) {
      return Left(ServerFailure(e.message ?? 'Error al crear cliente'));
    }
  }

  @override
  Future<Either<Failure, Client>> updateClient(Client client) async {
    try {
      final updated = await _datasource.updateClient(client);
      return Right(updated);
    } on FirebaseException catch (e) {
      return Left(ServerFailure(e.message ?? 'Error al actualizar'));
    }
  }

  @override
  Future<Either<Failure, void>> deleteClient(String id) async {
    try {
      await _datasource.deleteClient(id);
      return const Right(null);
    } on FirebaseException catch (e) {
      return Left(ServerFailure(e.message ?? 'Error al eliminar'));
    }
  }

  @override
  Future<Either<Failure, List<Client>>> searchClients(String query) async {
    try {
      final results = await _datasource.searchClients(query);
      return Right(results);
    } catch (e) {
      return Left(ServerFailure(e.toString()));
    }
  }

  @override
  Future<Either<Failure, String>> exportToCSV(List<Client> clients) async {
    try {
      final csv = await _datasource.exportToCSV(clients);
      return Right(csv);
    } catch (e) {
      return Left(ServerFailure('Error al exportar'));
    }
  }
}

---
# lib/features/clients/data/datasources/client_remote_datasource.dart
---

import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:uuid/uuid.dart';
import '../../../../core/constants/firestore_collections.dart';
import '../../domain/entities/client.dart';
import '../models/client_model.dart';

abstract class ClientRemoteDatasource {
  Stream<List<Client>> watchClients({
    ClientStatus? filterStatus,
    ClientCategory? filterCategory,
    String? searchQuery,
    int limit,
  });
  Future<Client?> getClientById(String id);
  Future<Client> createClient(Client client);
  Future<Client> updateClient(Client client);
  Future<void> deleteClient(String id);
  Future<List<Client>> searchClients(String query);
  Future<String> exportToCSV(List<Client> clients);
}

class ClientRemoteDatasourceImpl implements ClientRemoteDatasource {
  final FirebaseFirestore _db;
  final _uuid = const Uuid();

  ClientRemoteDatasourceImpl(this._db);

  CollectionReference get _col => _db.collection(FirestoreCollections.clients);

  @override
  Stream<List<Client>> watchClients({
    ClientStatus? filterStatus,
    ClientCategory? filterCategory,
    String? searchQuery,
    int limit = 50,
  }) {
    Query query = _col.orderBy('fechaAlta', descending: true).limit(limit);

    if (filterStatus != null) {
      query = query.where('estado', isEqualTo: filterStatus.name);
    }
    if (filterCategory != null) {
      query = query.where('categoria', isEqualTo: filterCategory.name);
    }
    if (searchQuery != null && searchQuery.isNotEmpty) {
      // Usa el array searchTerms pre-calculado en el modelo
      query = query.where('searchTerms',
          arrayContains: searchQuery.toLowerCase().trim());
    }

    return query.snapshots().map((snap) =>
        snap.docs.map((d) => ClientModel.fromFirestore(d)).toList());
  }

  @override
  Future<Client?> getClientById(String id) async {
    final doc = await _col.doc(id).get();
    if (!doc.exists) return null;
    return ClientModel.fromFirestore(doc);
  }

  @override
  Future<Client> createClient(Client client) async {
    final id = _uuid.v4();
    final model = ClientModel.fromEntity(client.copyWith());
    final data = (model as ClientModel).toFirestore();
    await _col.doc(id).set(data);
    return ClientModel.fromEntity(client);
  }

  @override
  Future<Client> updateClient(Client client) async {
    final model = ClientModel.fromEntity(client);
    await _col.doc(client.id).update({
      ...model.toFirestore(),
      'ultimaActividad': FieldValue.serverTimestamp(),
    });
    return client;
  }

  @override
  Future<void> deleteClient(String id) async {
    // Soft delete — preserva historial
    await _col.doc(id).update({
      'estado': 'eliminado',
      'deletedAt': FieldValue.serverTimestamp(),
    });
  }

  @override
  Future<List<Client>> searchClients(String query) async {
    final snap = await _col
        .where('searchTerms', arrayContains: query.toLowerCase())
        .limit(20)
        .get();
    return snap.docs.map((d) => ClientModel.fromFirestore(d)).toList();
  }

  @override
  Future<String> exportToCSV(List<Client> clients) async {
    final header = 'ID,Nombre,Apellido,Empresa,Email,Teléfono,CUIT,Categoría,Estado,Fecha Alta\n';
    final rows = clients.map((c) =>
      '"${c.id}","${c.nombre}","${c.apellido}","${c.empresa ?? ""}","${c.email ?? ""}","${c.telefono ?? ""}","${c.cuit ?? ""}","${c.categoria.name}","${c.estado.name}","${c.fechaAlta.toIso8601String()}"'
    ).join('\n');
    return header + rows;
  }
}

---
# lib/features/clients/presentation/providers/client_provider.dart
---

import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import '../../domain/entities/client.dart';
import '../../domain/repositories/client_repository.dart';
import '../../data/repositories/client_repository_impl.dart';
import '../../data/datasources/client_remote_datasource.dart';

// --- Infrastructure Providers ---

final firestoreProvider = Provider<FirebaseFirestore>(
  (_) => FirebaseFirestore.instance,
);

final clientDatasourceProvider = Provider<ClientRemoteDatasource>(
  (ref) => ClientRemoteDatasourceImpl(ref.watch(firestoreProvider)),
);

final clientRepositoryProvider = Provider<ClientRepository>(
  (ref) => ClientRepositoryImpl(ref.watch(clientDatasourceProvider)),
);

// --- Filter State ---

class ClientFilterState {
  final ClientStatus? status;
  final ClientCategory? category;
  final String? searchQuery;

  const ClientFilterState({this.status, this.category, this.searchQuery});

  ClientFilterState copyWith({
    ClientStatus? status, ClientCategory? category, String? searchQuery,
    bool clearStatus = false, bool clearCategory = false,
  }) => ClientFilterState(
    status: clearStatus ? null : status ?? this.status,
    category: clearCategory ? null : category ?? this.category,
    searchQuery: searchQuery ?? this.searchQuery,
  );
}

class ClientFilterNotifier extends StateNotifier<ClientFilterState> {
  ClientFilterNotifier() : super(const ClientFilterState());

  void setStatus(ClientStatus? status) =>
      state = state.copyWith(status: status, clearStatus: status == null);
  void setCategory(ClientCategory? category) =>
      state = state.copyWith(category: category, clearCategory: category == null);
  void setSearch(String? query) =>
      state = state.copyWith(searchQuery: query?.isEmpty == true ? null : query);
  void clear() => state = const ClientFilterState();
}

final clientFilterProvider = StateNotifierProvider<ClientFilterNotifier, ClientFilterState>(
  (_) => ClientFilterNotifier(),
);

// --- Clients Stream ---

final clientsStreamProvider = StreamProvider.autoDispose<List<Client>>((ref) {
  final repo = ref.watch(clientRepositoryProvider);
  final filter = ref.watch(clientFilterProvider);

  return repo
      .watchClients(
        filterStatus: filter.status,
        filterCategory: filter.category,
        searchQuery: filter.searchQuery,
      )
      .map((either) => either.fold(
            (failure) => throw Exception(failure.message),
            (clients) => clients,
          ));
});

// --- Single Client ---

final clientByIdProvider = FutureProvider.autoDispose.family<Client?, String>(
  (ref, id) async {
    final repo = ref.watch(clientRepositoryProvider);
    final result = await repo.getClientById(id);
    return result.fold((_) => null, (c) => c);
  },
);

// --- CRUD Actions ---

class ClientActionsNotifier extends StateNotifier<AsyncValue<void>> {
  final ClientRepository _repo;
  ClientActionsNotifier(this._repo) : super(const AsyncValue.data(null));

  Future<bool> create(Client client) async {
    state = const AsyncValue.loading();
    final result = await _repo.createClient(client);
    return result.fold(
      (failure) { state = AsyncValue.error(failure.message, StackTrace.current); return false; },
      (_) { state = const AsyncValue.data(null); return true; },
    );
  }

  Future<bool> update(Client client) async {
    state = const AsyncValue.loading();
    final result = await _repo.updateClient(client);
    return result.fold(
      (failure) { state = AsyncValue.error(failure.message, StackTrace.current); return false; },
      (_) { state = const AsyncValue.data(null); return true; },
    );
  }

  Future<bool> delete(String id) async {
    state = const AsyncValue.loading();
    final result = await _repo.deleteClient(id);
    return result.fold(
      (failure) { state = AsyncValue.error(failure.message, StackTrace.current); return false; },
      (_) { state = const AsyncValue.data(null); return true; },
    );
  }
}

final clientActionsProvider = StateNotifierProvider.autoDispose<ClientActionsNotifier, AsyncValue<void>>(
  (ref) => ClientActionsNotifier(ref.watch(clientRepositoryProvider)),
);

---
# lib/features/auth/presentation/providers/auth_provider.dart
---

import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../domain/entities/app_user.dart';

final firebaseAuthProvider = Provider<FirebaseAuth>(
  (_) => FirebaseAuth.instance,
);

final authStateProvider = StreamProvider<User?>((ref) {
  return ref.watch(firebaseAuthProvider).authStateChanges();
});

final currentUserProvider = FutureProvider<AppUser?>((ref) async {
  // Escucha cambios de auth y carga perfil de Firestore
  final authState = ref.watch(authStateProvider);
  return authState.when(
    data: (user) async {
      if (user == null) return null;
      // Cargar perfil desde Firestore
      final doc = await FirebaseFirestore.instance
          .collection('users').doc(user.uid).get();
      if (!doc.exists) return null;
      return UserModel.fromFirestore(doc);
    },
    loading: () => null,
    error: (_, __) => null,
  );
});

class AuthNotifier extends StateNotifier<AsyncValue<AppUser?>> {
  final FirebaseAuth _auth;
  final FirebaseFirestore _db;

  AuthNotifier(this._auth, this._db) : super(const AsyncValue.data(null));

  Future<bool> login(String email, String password) async {
    state = const AsyncValue.loading();
    try {
      final credential = await _auth.signInWithEmailAndPassword(
        email: email.trim(), password: password,
      );
      // Cargar perfil del usuario desde Firestore
      final doc = await _db.collection('users').doc(credential.user!.uid).get();
      final user = doc.exists ? UserModel.fromFirestore(doc) : null;
      state = AsyncValue.data(user);
      return true;
    } on FirebaseAuthException catch (e) {
      state = AsyncValue.error(_mapAuthError(e.code), StackTrace.current);
      return false;
    }
  }

  Future<bool> register({
    required String email,
    required String password,
    required String name,
    String? lastName,
  }) async {
    state = const AsyncValue.loading();
    try {
      final cred = await _auth.createUserWithEmailAndPassword(
        email: email.trim(), password: password,
      );
      final user = AppUser(
        id: cred.user!.uid,
        email: email.trim(),
        name: name,
        lastName: lastName,
        role: UserRole.vendedor,
        createdAt: DateTime.now(),
      );
      await _db.collection('users').doc(user.id).set(
        UserModel.fromEntity(user).toFirestore(),
      );
      state = AsyncValue.data(user);
      return true;
    } on FirebaseAuthException catch (e) {
      state = AsyncValue.error(_mapAuthError(e.code), StackTrace.current);
      return false;
    }
  }

  Future<void> logout() async {
    await _auth.signOut();
    state = const AsyncValue.data(null);
  }

  Future<bool> resetPassword(String email) async {
    try {
      await _auth.sendPasswordResetEmail(email: email.trim());
      return true;
    } on FirebaseAuthException {
      return false;
    }
  }

  String _mapAuthError(String code) => switch (code) {
    'user-not-found' => 'No existe una cuenta con ese email',
    'wrong-password' => 'Contraseña incorrecta',
    'email-already-in-use' => 'El email ya está registrado',
    'weak-password' => 'La contraseña es demasiado débil',
    'invalid-email' => 'Email inválido',
    'too-many-requests' => 'Demasiados intentos. Intente más tarde',
    _ => 'Error de autenticación',
  };
}

final authNotifierProvider = StateNotifierProvider<AuthNotifier, AsyncValue<AppUser?>>(
  (ref) => AuthNotifier(
    ref.watch(firebaseAuthProvider),
    ref.watch(firestoreProvider),
  ),
);
