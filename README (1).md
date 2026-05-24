# ![alt text(image.png)] RapiKids

**Guarderías cercanas y certificadas — Bogotá, Colombia**

RapiKids es una aplicación móvil Android que conecta a padres y acudientes con guarderías infantiles verificadas, permitiendo buscarlas por ubicación, consultar perfiles detallados, calificarlas y contactarlas directamente.

[![Android](https://img.shields.io/badge/Platform-Android%208.0%2B-brightgreen?logo=android)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Language-Kotlin-blueviolet?logo=kotlin)](https://kotlinlang.org)
[![Jetpack Compose](https://img.shields.io/badge/UI-Jetpack%20Compose-blue?logo=jetpackcompose)](https://developer.android.com/jetpack/compose)
[![Supabase](https://img.shields.io/badge/Backend-Supabase-3ECF8E?logo=supabase)](https://supabase.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📑 Tabla de contenido

- [Descripción general](#-descripción-general)
- [Arquitectura del sistema](#-arquitectura-del-sistema)
- [Tecnologías y dependencias](#-tecnologías-y-dependencias)
- [Estructura del proyecto](#-estructura-del-proyecto)
- [Requisitos previos](#-requisitos-previos)
- [Configuración de Supabase](#-configuración-de-supabase)
- [Instalación y ejecución](#-instalación-y-ejecución)
- [Variables de entorno y configuración](#-variables-de-entorno-y-configuración)
- [Módulos del sistema](#-módulos-del-sistema)
- [Base de datos](#-base-de-datos)
- [Capturas de pantalla](#-capturas-de-pantalla)
- [Solución de problemas](#-solución-de-problemas)
- [Contribuir](#-contribuir)

---

## 📱 Descripción general

RapiKids resuelve el problema de encontrar guarderías infantiles confiables en Bogotá. La plataforma conecta tres actores:

| Rol | Descripción |
|-----|-------------|
| **Padre / Acudiente** | Busca guarderías verificadas, filtra por ubicación, consulta perfiles y deja reseñas |
| **Guardería** | Registra su institución, sube documentos de verificación y gestiona su perfil público |
| **Administrador** | Revisa solicitudes, verifica o rechaza guarderías con mensaje explicativo |

**Características principales:**
- Búsqueda en tiempo real por nombre o dirección
- Mapa interactivo con OpenStreetMap (MapLibre) — sin costos de API
- Cálculo de distancias con la fórmula de Haversine
- Registro con marcación de ubicación exacta en mapa
- Sistema de calificaciones con estrellas y comentarios
- Verificación de correo electrónico al registrarse
- Preguntas frecuentes por rol
- Splash screen con logo de la aplicación

---

## 🏗 Arquitectura del sistema

El proyecto sigue el patrón **MVVM (Model-View-ViewModel)** con las siguientes capas:

```
┌─────────────────────────────────────────────────────────────┐
│  CAPA 1 — USUARIOS                                          │
│  Padre/Acudiente · Guardería · Administrador                │
└────────────────────────┬────────────────────────────────────┘
                         │ interacción UI
┌────────────────────────▼────────────────────────────────────┐
│  CAPA 2 — VISTA (Jetpack Compose)                           │
│  LoginScreen · HomePadreScreen · HomeGuarderiaScreen        │
│  HomeAdminScreen · PerfilGuarderiaScreen · RegisterScreens  │
└────────────────────────┬────────────────────────────────────┘
                         │ StateFlow / collectAsState
┌────────────────────────▼────────────────────────────────────┐
│  CAPA 3 — VIEWMODEL                                         │
│  AuthViewModel · HomePadreViewModel · HomeGuarderiaViewModel│
│  AdminViewModel · PerfilGuarderiaViewModel                  │
└────────────────────────┬────────────────────────────────────┘
                         │ suspend fun / coroutines
┌────────────────────────▼────────────────────────────────────┐
│  CAPA 4 — REPOSITORIOS                                      │
│  AuthRepository · GuarderiaRepository · AdminRepository     │
│  GuarderiaProfileRepository · LocationService               │
└────────────────────────┬────────────────────────────────────┘
                         │ SDK / HTTP
┌────────────────────────▼────────────────────────────────────┐
│  CAPA 5 — SUPABASE (Backend)                                │
│  Auth (JWT) · PostgreSQL (postgrest) · Storage              │
└────────────────────────┬────────────────────────────────────┘
                         │ REST / GPS
┌────────────────────────▼────────────────────────────────────┐
│  CAPA 6 — SERVICIOS EXTERNOS                                │
│  MapLibre + OpenStreetMap · Nominatim · Google Play Services│
└─────────────────────────────────────────────────────────────┘
```

### Flujo de datos
- Las **Vistas** observan `StateFlow` expuestos por los **ViewModels**.
- Los **ViewModels** delegan la lógica de negocio a los **Repositorios**.
- Los **Repositorios** se comunican con **Supabase** vía el SDK oficial de Kotlin.
- Las coordenadas geográficas se almacenan en la base de datos al momento del registro, eliminando la dependencia de geocodificación en tiempo real.

---

## 🛠 Tecnologías y dependencias

### Núcleo
| Dependencia | Versión | Uso |
|-------------|---------|-----|
| Kotlin | 1.9.x | Lenguaje principal |
| Jetpack Compose BOM | 2024.x | UI declarativa |
| Navigation Compose | — | Navegación entre pantallas |
| Lifecycle ViewModel | — | Gestión de estado |
| Coroutines | — | Operaciones asíncronas |

### Backend
| Dependencia | Versión | Uso |
|-------------|---------|-----|
| Supabase BOM | 3.1.4 | Backend completo |
| supabase-kt auth | — | Autenticación JWT |
| supabase-kt postgrest | — | Acceso a base de datos |
| supabase-kt storage | — | Almacenamiento de archivos |
| Ktor Client Android | 3.1.3 | Cliente HTTP |

### Mapas y ubicación
| Dependencia | Versión | Uso |
|-------------|---------|-----|
| MapLibre Android SDK | 11.5.2 | Renderizado de mapas |
| Google Play Services Location | 21.2.0 | GPS con FusedLocationClient |
| OpenStreetMap / Nominatim | — | Tiles y geocodificación gratuita |

### UI
| Dependencia | Versión | Uso |
|-------------|---------|-----|
| Coil Compose | 2.6.0 | Carga de imágenes asíncronas |
| Material Icons Extended | — | Iconografía |
| Kotlinx Serialization | 1.7.3 | Serialización JSON |

### En `build.gradle.kts` (módulo app):
```kotlin
implementation(platform("io.github.jan-tennert.supabase:bom:3.1.4"))
implementation("io.github.jan-tennert.supabase:auth-kt")
implementation("io.github.jan-tennert.supabase:postgrest-kt")
implementation("io.github.jan-tennert.supabase:storage-kt")
implementation("io.ktor:ktor-client-android:3.1.3")
implementation("io.coil-kt:coil-compose:2.6.0")
implementation("org.maplibre.gl:android-sdk:11.5.2")
implementation("com.google.android.gms:play-services-location:21.2.0")
implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
implementation("androidx.compose.material:material-icons-extended")
implementation("androidx.core:core-splashscreen:1.0.1")
```

---

## 📁 Estructura del proyecto

```
app/src/main/java/com/example/rapikids01/
│
├── data/
│   ├── auth/
│   │   └── AuthRepository.kt          # Registro, login, logout
│   ├── location/
│   │   └── LocationService.kt         # GPS + fórmula Haversine
│   ├── model/
│   │   └── Models.kt                  # Guarderia, Usuario, Admin, Resena
│   ├── repository/
│   │   ├── AdminRepository.kt         # CRUD administrador
│   │   ├── GuarderiaProfileRepository.kt  # Perfil y fotos de guardería
│   │   └── GuarderiaRepository.kt     # Consultas públicas de guarderías
│   └── supabase/
│       └── SupabaseClient.kt          # Instancia singleton del cliente
│
├── navigation/
│   └── Routes.kt                      # Constantes de navegación
│
├── screen/
│   ├── AppNavigation.kt               # NavHost principal
│   ├── home/
│   │   ├── Home.kt                    # Selector de rol
│   │   ├── HomeAdminScreen.kt         # Panel de administrador
│   │   ├── HomeGuarderiaScreen.kt     # Perfil y gestión de guardería
│   │   ├── HomePadreScreen.kt         # Búsqueda y lista de guarderías
│   │   ├── MapaGuarderiasScreen.kt    # Mapa interactivo
│   │   ├── PerfilGuarderiaScreen.kt   # Detalle + calificaciones
│   │   └── FaqScreen.kt              # Preguntas frecuentes por rol
│   ├── login/
│   │   └── LoginScreen.kt             # Login compartido (3 roles)
│   └── register/
│       ├── RegisterAdminScreen.kt
│       ├── RegisterGuarderiaScreen.kt
│       └── RegisterPadreScreen.kt
│
├── viewmodel/
│   ├── AdminViewModel.kt
│   ├── AuthViewModel.kt
│   ├── HomeGuarderiaViewModel.kt
│   └── HomePadreViewModel.kt
│
├── ui/theme/                          # Colores, tipografía, tema
├── MainActivity.kt                    # Entry point + SplashScreen
└── UserRole.kt                        # Enum de roles
```

---

## ✅ Requisitos previos

Antes de clonar y ejecutar el proyecto asegúrese de tener instalado:

- **Android Studio** Hedgehog (2023.1.1) o superior
- **JDK 11** o superior
- **Android SDK** con API 26 (mínimo) y API 36 (recomendado)
- Cuenta en [Supabase](https://supabase.com) (gratuita)
- Dispositivo físico o emulador con Android 8.0+

---

## ⚙️ Configuración de Supabase

### 1. Crear proyecto en Supabase
1. Ingrese a [supabase.com](https://supabase.com) y cree un nuevo proyecto.
2. Anote la **Project URL** y la **Anon Public Key** (Settings → API).

### 2. Crear las tablas en SQL Editor

Ejecute el siguiente SQL en **Supabase → SQL Editor**:

```sql
-- Tabla de usuarios (todos los roles)
CREATE TABLE public.users (
  uid       UUID PRIMARY KEY,
  email     TEXT NOT NULL,
  nombre    TEXT NOT NULL DEFAULT '',
  telefono  TEXT NOT NULL DEFAULT '',
  role      TEXT NOT NULL CHECK (role IN ('PADRE', 'GUARDERIA', 'ADMIN')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla de guarderías
CREATE TABLE public.guarderias (
  uid                UUID PRIMARY KEY,
  email              TEXT NOT NULL,
  nombre_guarderia   TEXT NOT NULL DEFAULT '',
  direccion          TEXT NOT NULL DEFAULT '',
  nit                TEXT NOT NULL DEFAULT '',
  telefono           TEXT NOT NULL DEFAULT '',
  foto_url           TEXT DEFAULT '',
  documento_url      TEXT DEFAULT '',
  verificada         BOOLEAN DEFAULT false,
  estado             TEXT DEFAULT 'pendiente',
  mensaje_rechazo    TEXT DEFAULT '',
  calificacion_promedio DOUBLE PRECISION DEFAULT 0.0,
  total_resenas      INTEGER DEFAULT 0,
  latitud            DOUBLE PRECISION,
  longitud           DOUBLE PRECISION,
  descripcion        TEXT DEFAULT '',
  precio_mensual     INTEGER DEFAULT 0,
  hora_apertura      TEXT DEFAULT '',
  hora_cierre        TEXT DEFAULT '',
  dias_atencion      TEXT DEFAULT '',
  jornada            TEXT DEFAULT '',
  fotos              TEXT[] DEFAULT '{}',
  created_at         TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla de reseñas
CREATE TABLE public.resenas (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  guarderia_uid UUID NOT NULL REFERENCES public.guarderias(uid),
  padre_uid    UUID NOT NULL,
  calificacion INTEGER NOT NULL CHECK (calificacion BETWEEN 1 AND 5),
  comentario   TEXT NOT NULL DEFAULT '',
  nombre_padre TEXT DEFAULT '',
  created_at   TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla de admins
CREATE TABLE public.admins (
  uid       UUID PRIMARY KEY,
  email     TEXT NOT NULL,
  nombre    TEXT NOT NULL DEFAULT '',
  cargo     TEXT NOT NULL DEFAULT '',
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 3. Crear trigger para registro con verificación de correo

Cuando el usuario confirma su correo, este trigger crea automáticamente su registro en `public.users`:

```sql
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS trigger AS $$
DECLARE
  meta jsonb;
  rol  text;
BEGIN
  meta := NEW.raw_user_meta_data;
  rol  := meta->>'role';

  INSERT INTO public.users (uid, email, nombre, telefono, role)
  VALUES (NEW.id, NEW.email,
    COALESCE(meta->>'nombre', ''),
    COALESCE(meta->>'telefono', ''),
    COALESCE(rol, 'PADRE'))
  ON CONFLICT (uid) DO NOTHING;

  IF rol = 'GUARDERIA' THEN
    INSERT INTO public.guarderias (
      uid, email, nombre_guarderia, direccion, nit, telefono,
      documento_url, verificada, estado, descripcion,
      precio_mensual, hora_apertura, hora_cierre, dias_atencion, jornada,
      latitud, longitud)
    VALUES (
      NEW.id, NEW.email,
      COALESCE(meta->>'nombre', ''),
      COALESCE(meta->>'direccion', ''),
      COALESCE(meta->>'nit', ''),
      COALESCE(meta->>'telefono', ''),
      COALESCE(meta->>'documento_url', ''),
      false, 'pendiente', '', 0, '', '', '', '',
      CASE WHEN meta->>'latitud'  IS NOT NULL THEN (meta->>'latitud')::double precision  ELSE NULL END,
      CASE WHEN meta->>'longitud' IS NOT NULL THEN (meta->>'longitud')::double precision ELSE NULL END)
    ON CONFLICT (uid) DO NOTHING;
  END IF;

  IF rol = 'ADMIN' THEN
    INSERT INTO public.admins (uid, email, nombre, cargo)
    VALUES (NEW.id, NEW.email,
      COALESCE(meta->>'nombre', ''),
      COALESCE(meta->>'cargo', ''))
    ON CONFLICT (uid) DO NOTHING;
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE OR REPLACE TRIGGER on_auth_user_confirmed
  AFTER UPDATE ON auth.users
  FOR EACH ROW
  WHEN (OLD.email_confirmed_at IS NULL AND NEW.email_confirmed_at IS NOT NULL)
  EXECUTE FUNCTION public.handle_new_user();
```

### 4. Crear bucket de Storage

En **Supabase → Storage**, cree un bucket llamado `guarderias` con acceso **público**.

### 5. Activar verificación de correo

En **Authentication → Providers → Email** active **"Confirm email"**.

---

## 🚀 Instalación y ejecución

### Clonar el repositorio

```bash
git clone https://github.com/Juan-montoya-22/Rapikids-0.1.git
cd Rapikids-0.1
```

### Configurar credenciales de Supabase

Abra el archivo `app/src/main/java/com/example/rapikids01/data/supabase/SupabaseClient.kt` y reemplace los valores:

```kotlin
val client = createSupabaseClient(
    supabaseUrl = "https://TU_PROJECT_ID.supabase.co",  // ← Su Project URL
    supabaseKey = "TU_ANON_PUBLIC_KEY"                   // ← Su Anon Key
) {
    install(Auth)
    install(Postgrest)
    install(Storage)
}
```

### Abrir en Android Studio

1. Abra **Android Studio**.
2. Seleccione **File → Open** y navegue hasta la carpeta del proyecto.
3. Espere a que Gradle sincronice las dependencias (puede tardar unos minutos la primera vez).
4. Conecte un dispositivo Android o inicie un emulador con API 26+.
5. Presione **▶ Run** o use `Shift + F10`.

### Compilar APK de debug

```bash
./gradlew assembleDebug
```

El APK generado estará en:
```
app/build/outputs/apk/debug/app-debug.apk
```

### Compilar APK de release

```bash
./gradlew assembleRelease
```

---

## 🔑 Variables de entorno y configuración

| Constante | Archivo | Descripción |
|-----------|---------|-------------|
| `supabaseUrl` | `SupabaseClient.kt` | URL del proyecto Supabase |
| `supabaseKey` | `SupabaseClient.kt` | Clave anon pública de Supabase |
| `ADMIN_SECRET_CODE` | `AuthRepository.kt` | Código secreto para registro de admins |

> ⚠️ **Importante:** No suba las claves de Supabase al repositorio público. Considere usar un archivo `local.properties` o variables de entorno de Gradle para producción.

### Ejemplo con `local.properties` (recomendado para producción)

En `local.properties`:
```properties
supabase.url=https://tu_proyecto.supabase.co
supabase.key=tu_anon_key
```

En `build.gradle.kts`:
```kotlin
val localProperties = java.util.Properties()
localProperties.load(rootProject.file("local.properties").inputStream())

android {
    defaultConfig {
        buildConfigField("String", "SUPABASE_URL", "\"${localProperties["supabase.url"]}\"")
        buildConfigField("String", "SUPABASE_KEY", "\"${localProperties["supabase.key"]}\"")
    }
}
```

---

## 📦 Módulos del sistema

### Autenticación (`AuthRepository.kt`)
- Registro por rol con metadatos en Supabase Auth
- Login con detección automática de rol
- Verificación de correo electrónico obligatoria
- Logout con limpieza de sesión

### Módulo padre (`HomePadreViewModel.kt`)
- Carga de guarderías verificadas desde Supabase
- Filtro por texto en tiempo real
- Geolocalización con FusedLocationClient
- Cálculo de distancias con fórmula de Haversine
- Filtro de guarderías a menos de 3 km

### Módulo guardería (`HomeGuarderiaViewModel.kt` + `GuarderiaProfileRepository.kt`)
- Carga y edición del perfil institucional
- Subida de foto de perfil y carrusel (hasta 3 fotos) a Supabase Storage
- Resubida de documentos en caso de rechazo

### Módulo administrador (`AdminViewModel.kt` + `AdminRepository.kt`)
- Lista de guarderías pendientes y todas
- Aprobación y rechazo con mensaje
- Panel de estadísticas en tiempo real

### Mapa interactivo (`MapaGuarderiasScreen.kt`)
- Renderizado con MapLibre + tiles de OpenFreemap
- Marcador personalizado para ubicación del usuario
- Marcadores de guarderías con tarjeta de detalle al seleccionar
- Recarga dinámica al cambiar la lista

---

## 🗄 Base de datos

### Diagrama de tablas

```
auth.users (Supabase interno)
    │
    ├──► public.users
    │       uid, email, nombre, telefono, role, created_at
    │
    ├──► public.guarderias
    │       uid, email, nombre_guarderia, direccion, nit,
    │       telefono, foto_url, documento_url, verificada,
    │       estado, mensaje_rechazo, calificacion_promedio,
    │       total_resenas, latitud, longitud, descripcion,
    │       precio_mensual, hora_apertura, hora_cierre,
    │       dias_atencion, jornada, fotos[], created_at
    │
    ├──► public.resenas
    │       id, guarderia_uid → guarderias.uid,
    │       padre_uid, calificacion, comentario,
    │       nombre_padre, created_at
    │
    └──► public.admins
            uid, email, nombre, cargo, created_at
```

### Estados de guardería

| Estado | Descripción |
|--------|-------------|
| `pendiente` | Solicitud enviada, en revisión por el administrador |
| `verificada` | Aprobada, visible en la búsqueda de acudientes |
| `rechazada` | Rechazada con motivo, puede reenviar documento |

---

## 🔧 Solución de problemas

### Error: `NoSuchMethodError` al iniciar la app
**Causa:** Caché de compilación desactualizado.  
**Solución:** En Android Studio: `Build → Clean Project` → `File → Invalidate Caches → Invalidate and Restart` → `Build → Rebuild Project`.

### Error: `JAVA_HOME is not set`
**Causa:** La terminal del sistema no tiene Java configurado.  
**Solución:** Use los menús de Android Studio (`Build → Clean Project`) en lugar de la terminal del sistema.

### El mapa no muestra los tiles
**Causa:** Sin conexión a Internet o URL de estilo no disponible.  
**Solución:** Verifique la conexión. El estilo usado es `https://tiles.openfreemap.org/styles/liberty`.

### La guardería no aparece en el mapa tras registrarse
**Causa:** Las coordenadas no se guardaron o el estado no es `verificada`.  
**Solución:** Verifique en Supabase que `latitud` y `longitud` no sean `null` y que `estado = 'verificada'`.

### El trigger no crea el usuario en `public.users`
**Causa:** El trigger solo se dispara al confirmar el correo.  
**Solución:** Asegúrese de que el usuario haya verificado su correo. Compruebe que el trigger `on_auth_user_confirmed` esté activo en `Database → Triggers`.

---

## 📄 Desarrolllado 

Este proyecto fue desarrollado como trabajo de grado en la **Universidad Antonio Nariño** — Facultad de Ingeniería de sistema.
---

<p align="center">
  ![alt text](image-1.png) <strong>RapiKids</strong> · Bogotá, Colombia · 2025
</p>
