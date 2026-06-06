# ═══════════════════════════════════════════════
# lib/core/theme/app_theme.dart
# ═══════════════════════════════════════════════

import 'package:flutter/material.dart';
import 'app_colors.dart';

class AppTheme {
  AppTheme._();

  static ThemeData get light => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.primary,
      brightness: Brightness.light,
      primary: AppColors.primary,
      secondary: AppColors.secondary,
      surface: AppColors.surfaceLight,
      background: AppColors.backgroundLight,
      error: AppColors.error,
    ),
    fontFamily: 'DM_Sans',
    scaffoldBackgroundColor: AppColors.backgroundLight,
    cardTheme: CardTheme(
      elevation: 0,
      color: AppColors.surfaceLight,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
        side: BorderSide(color: AppColors.borderLight, width: 1),
      ),
    ),
    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: AppColors.surfaceLight,
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8),
        borderSide: BorderSide(color: AppColors.borderLight),
      ),
      enabledBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8),
        borderSide: BorderSide(color: AppColors.borderLight),
      ),
      focusedBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8),
        borderSide: BorderSide(color: AppColors.primary, width: 1.5),
      ),
      errorBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8),
        borderSide: BorderSide(color: AppColors.error),
      ),
      contentPadding: const EdgeInsets.symmetric(horizontal: 14, vertical: 12),
      hintStyle: TextStyle(color: AppColors.textTertiary, fontSize: 14),
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: AppColors.primary,
        foregroundColor: Colors.white,
        elevation: 0,
        padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
        textStyle: const TextStyle(fontSize: 13, fontWeight: FontWeight.w500),
      ),
    ),
    textButtonTheme: TextButtonThemeData(
      style: TextButton.styleFrom(
        foregroundColor: AppColors.primary,
        textStyle: const TextStyle(fontSize: 13, fontWeight: FontWeight.w500),
      ),
    ),
    outlinedButtonTheme: OutlinedButtonThemeData(
      style: OutlinedButton.styleFrom(
        foregroundColor: AppColors.textSecondary,
        side: BorderSide(color: AppColors.borderLight),
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 10),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
      ),
    ),
    chipTheme: ChipThemeData(
      backgroundColor: AppColors.backgroundLight,
      selectedColor: AppColors.primaryLight,
      labelStyle: const TextStyle(fontSize: 12),
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
      side: BorderSide(color: AppColors.borderLight),
    ),
    dividerTheme: DividerThemeData(
      color: AppColors.borderLight,
      thickness: 1,
      space: 0,
    ),
    appBarTheme: AppBarTheme(
      backgroundColor: AppColors.surfaceLight,
      elevation: 0,
      centerTitle: false,
      titleTextStyle: TextStyle(
        fontSize: 16, fontWeight: FontWeight.w600,
        color: AppColors.textPrimary, fontFamily: 'DM_Sans',
      ),
      iconTheme: IconThemeData(color: AppColors.textSecondary),
    ),
    tooltipTheme: TooltipThemeData(
      decoration: BoxDecoration(
        color: AppColors.textPrimary,
        borderRadius: BorderRadius.circular(6),
      ),
      textStyle: const TextStyle(color: Colors.white, fontSize: 12),
    ),
  );

  static ThemeData get dark => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.primary,
      brightness: Brightness.dark,
      primary: AppColors.primaryLight,
      secondary: AppColors.secondary,
      surface: AppColors.surfaceDark,
      background: AppColors.backgroundDark,
      error: AppColors.error,
    ),
    fontFamily: 'DM_Sans',
    scaffoldBackgroundColor: AppColors.backgroundDark,
    cardTheme: CardTheme(
      elevation: 0,
      color: AppColors.surfaceDark,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
        side: BorderSide(color: AppColors.borderDark, width: 1),
      ),
    ),
    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: AppColors.surfaceDark2,
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8),
        borderSide: BorderSide(color: AppColors.borderDark),
      ),
      enabledBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8),
        borderSide: BorderSide(color: AppColors.borderDark),
      ),
      focusedBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8),
        borderSide: BorderSide(color: AppColors.primaryLight, width: 1.5),
      ),
      errorBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8),
        borderSide: BorderSide(color: AppColors.error),
      ),
      contentPadding: const EdgeInsets.symmetric(horizontal: 14, vertical: 12),
      hintStyle: TextStyle(color: AppColors.textTertiaryDark, fontSize: 14),
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: AppColors.primaryLight,
        foregroundColor: Colors.white,
        elevation: 0,
        padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
      ),
    ),
  );
}

---
# ═══════════════════════════════════════════════
# lib/core/constants/app_colors.dart
# ═══════════════════════════════════════════════

import 'package:flutter/material.dart';

class AppColors {
  AppColors._();

  // Brand
  static const Color primary = Color(0xFF2563EB);
  static const Color primaryLight = Color(0xFF4A7FFF);
  static const Color primaryGlow = Color(0x264A7FFF);
  static const Color secondary = Color(0xFF9B6DFF);

  // Status
  static const Color success = Color(0xFF22C97A);
  static const Color successGlow = Color(0x1F22C97A);
  static const Color warning = Color(0xFFF5A623);
  static const Color warningGlow = Color(0x1FF5A623);
  static const Color error = Color(0xFFF04060);
  static const Color errorGlow = Color(0x1AF04060);
  static const Color info = Color(0xFF3B82F6);

  // Light Mode
  static const Color backgroundLight = Color(0xFFF8FAFC);
  static const Color surfaceLight = Color(0xFFFFFFFF);
  static const Color borderLight = Color(0xFFE2E8F0);
  static const Color textPrimary = Color(0xFF0F172A);
  static const Color textSecondary = Color(0xFF475569);
  static const Color textTertiary = Color(0xFF94A3B8);

  // Dark Mode
  static const Color backgroundDark = Color(0xFF0F1117);
  static const Color surfaceDark = Color(0xFF161B27);
  static const Color surfaceDark2 = Color(0xFF1E2535);
  static const Color surfaceDark3 = Color(0xFF252D3D);
  static const Color borderDark = Color(0xFF2A3347);
  static const Color borderDark2 = Color(0xFF344060);
  static const Color textPrimaryDark = Color(0xFFE2E8F4);
  static const Color textSecondaryDark = Color(0xFF8B9AB5);
  static const Color textTertiaryDark = Color(0xFF5C6B87);

  // Categorical (for charts, avatars)
  static const List<Color> categorical = [
    Color(0xFF3B82F6), Color(0xFF6366F1), Color(0xFF8B5CF6),
    Color(0xFF22C55E), Color(0xFFF59E0B), Color(0xFFEF4444),
    Color(0xFF14B8A6), Color(0xFFF97316),
  ];

  static Color categoricalForString(String seed) {
    int hash = 0;
    for (final char in seed.codeUnits) {
      hash = char + ((hash << 5) - hash);
    }
    return categorical[hash.abs() % categorical.length];
  }
}

---
# ═══════════════════════════════════════════════
# lib/core/constants/firestore_collections.dart
# ═══════════════════════════════════════════════

class FirestoreCollections {
  FirestoreCollections._();

  static const String users = 'users';
  static const String clients = 'clients';
  static const String leads = 'leads';
  static const String tasks = 'tasks';
  static const String deals = 'deals';
  static const String activities = 'activities';
  static const String reports = 'reports';

  // Subcollections
  static const String clientActivities = 'activities';
  static const String leadHistory = 'history';
  static const String dealNotes = 'notes';
}

---
# ═══════════════════════════════════════════════
# lib/core/utils/validators.dart
# ═══════════════════════════════════════════════

class Validators {
  Validators._();

  static String? required(String? value, [String? fieldName]) {
    if (value == null || value.trim().isEmpty) {
      return '${fieldName ?? "Este campo"} es requerido';
    }
    return null;
  }

  static String? email(String? value) {
    if (value == null || value.isEmpty) return 'El email es requerido';
    final regex = RegExp(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$');
    if (!regex.hasMatch(value)) return 'Email inválido';
    return null;
  }

  static String? password(String? value) {
    if (value == null || value.isEmpty) return 'La contraseña es requerida';
    if (value.length < 8) return 'Mínimo 8 caracteres';
    if (!value.contains(RegExp(r'[A-Z]'))) return 'Debe tener al menos una mayúscula';
    if (!value.contains(RegExp(r'[0-9]'))) return 'Debe tener al menos un número';
    return null;
  }

  static String? cuit(String? value) {
    if (value == null || value.isEmpty) return null; // opcional
    final clean = value.replaceAll(RegExp(r'[-\s]'), '');
    if (clean.length != 11) return 'CUIT inválido (ej: 20-12345678-9)';
    if (!RegExp(r'^\d+$').hasMatch(clean)) return 'CUIT debe contener solo números';
    return null;
  }

  static String? phone(String? value) {
    if (value == null || value.isEmpty) return null; // opcional
    final clean = value.replaceAll(RegExp(r'[\s\-\(\)\+]'), '');
    if (clean.length < 8 || clean.length > 15) return 'Teléfono inválido';
    return null;
  }

  static String? positiveNumber(String? value) {
    if (value == null || value.isEmpty) return null;
    final n = double.tryParse(value);
    if (n == null) return 'Debe ser un número';
    if (n < 0) return 'Debe ser mayor a 0';
    return null;
  }
}
