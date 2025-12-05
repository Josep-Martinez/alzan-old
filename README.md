# Alzan · Plataforma de Fitness y Nutrición 💪🥗

<div align="center">

**Aplicación móvil multiplataforma que unifica planificación deportiva, seguimiento nutricional y análisis de progreso con inteligencia artificial.**

[![Expo](https://img.shields.io/badge/Expo-54.0-000020?style=flat&logo=expo)](https://expo.dev)
[![React Native](https://img.shields.io/badge/React_Native-0.81-61DAFB?style=flat&logo=react)](https://reactnative.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?style=flat&logo=typescript)](https://www.typescriptlang.org)
[![License](https://img.shields.io/badge/License-Private-red)](./LICENSE)

[Características](#-características-principales) •
[Instalación](#-instalación-rápida) •
[Documentación](#-documentación) •
[Contribuir](#-contribuir) •
[Roadmap](#-roadmap)

</div>

---

## 🎯 Características Principales

### 🏠 Dashboard Motivacional
- Tarjetas de resumen diario (calorías, pasos, hábitos)
- Progreso semanal visual con indicadores de consistencia
- Accesos rápidos a entrenamientos y registro de comidas
- Widgets personalizables

### 💪 Planificador Deportivo Multi-Deporte
- **Gimnasio**: Gestión de ejercicios, series, repeticiones, peso y superseries
- **Running**: Distancia, ritmo, elevación y tipos de entrenamiento
- **Ciclismo**: Velocidad, potencia, cadencia y terreno
- **Natación**: Estilos, largos y métricas de piscina
- Timer de descanso inteligente con vibración
- Registro de intensidad (RPE) post-entrenamiento

### 🥗 Seguimiento Nutricional Inteligente
- Agenda de comidas (desayuno, almuerzo, cena, snacks)
- **Análisis con IA**: Escanea fotos de comida con Google Gemini para detectar nutrientes automáticamente
- **Scanner de código de barras**: Integración con OpenFoodFacts
- Cálculo automático de macros (proteínas, carbohidratos, grasas)
- Control de hidratación
- Clasificación de alimentos (real, procesada, ultraprocesada)

### 📊 Panel de Estadísticas
- Progreso semanal/mensual de calorías y macros
- Récords personales (PRs) por ejercicio
- Tracker de consistencia con heatmap visual
- Resumen de entrenamientos por tipo
- Exportación de datos a Excel

### 🎨 Diseño Premium
- Interfaz con gradientes vibrantes y dark mode
- Animaciones fluidas con `react-native-reanimated`
- Efectos glassmorphism con  `expo-blur`
- Feedback táctil con `expo-haptics`
- Iconografía de Material Design

---

## 🏗️ Stack Tecnológico

| Categoría | Tecnologías |
|-----------|------------|
| **Framework** | Expo 54, React Native 0.81, React 19 |
| **Lenguaje** | TypeScript 5.9 (strict mode) |
| **Routing** | expo-router (file-based) |
| **UI/UX** | react-native-paper, @expo/vector-icons, expo-linear-gradient |
| **Estado** | React Hooks, Context API |
| **Cámara/Medios** | expo-camera, expo-image-picker, expo-image-manipulator |
| **IA** | Google Gemini 2.0 Flash (Vision API) |
| **Servicios** | OpenFoodFacts API, expo-haptics, expo-localization |
| **Utilidades** | xlsx (exportación), react-native-gesture-handler |

---

## 🚀 Instalación Rápida

### Requisitos Previos

- **Node.js** 18+ (LTS recomendado)
- **pnpm** 10.24+ (o npm/yarn)
- **Expo Go** app (iOS/Android) para testing rápido
- **Xcode** (macOS) o **Android Studio** para emuladores (opcional)

### Pasos

```bash
# 1. Clonar repositorio
git clone https://github.com/tu-usuario/alzan.git
cd alzan

# 2. Instalar dependencias (usa pnpm si está configurado)
pnpm install
# o
npm install

# 3. Configurar variables de entorno
cp .env.example .env
# Editar .env y añadir tu GEMINI_API_KEY

# 4. Iniciar servidor de desarrollo
pnpm start
```

**Opciones de ejecución**:
- Presiona **`i`** para iOS Simulator
- Presiona **`a`** para Android Emulator  
- Presiona **`w`** para Web Browser
- Escanea QR con **Expo Go** app en tu teléfono

---

## 📖 Documentación

La documentación completa está organizada en la carpeta [`docs/`](./docs):

| Documento | Descripción |
|-----------|-------------|
| [**ARCHITECTURE.md**](./docs/ARCHITECTURE.md) | Arquitectura del proyecto, stack técnico, flujo de datos y patrones de diseño |
| [**COMPONENTS.md**](./docs/COMPONENTS.md) | Guía detallada de todos los componentes (props, uso, ejemplos) |
| [**SERVICES.md**](./docs/SERVICES.md) | Documentación de servicios e integraciones (Gemini, OpenFoodFacts, storage) |
| [**CONTRIBUTING.md**](./docs/CONTRIBUTING.md) | Guía para contribuir (setup, convenciones, pull requests) |
| [**DEPLOYMENT.md**](./docs/DEPLOYMENT.md) | Instrucciones para builds y despliegue en App Store/Play Store |

### Diagramas de Arquitectura

**Navegación entre pantallas**:
```
🏠 Dashboard → 💪 Deportes → 📋 Sesión Activa → 🎯 Intensidad
           → 🥗 Nutrición → 📸 Análisis IA → ✅ Confirmar
           → 📊 Estadísticas
```

**Flujo de análisis con IA**:
```
Usuario toma foto → GeminiFoodAnalyzer → Gemini API → JSON Response → Validación → Pre-fill Form → Confirmar
```

---

## 📁 Estructura del Proyecto

```
alzan-old/
├── app/                          # Rutas Expo Router
│   ├── (tabs)/                   # Tab Navigator
│   │   ├── index.tsx             # 🏠 Dashboard
│   │   ├── work.tsx              # 💪 Planificador Deportivo
│   │   ├── nutrition.tsx         # 🥗 Seguimiento Nutricional
│   │   └── stats.tsx             # 📊 Panel de Estadísticas
│   ├── _layout.tsx               # Layout raíz
│   └── +not-found.tsx            # Página 404
│
├── components/
│   ├── sport/                    # Componentes deportivos
│   │   ├── gym/                  # Gimnasio (sesiones, ejercicios, timers)
│   │   ├── other/                # Running, ciclismo, natación
│   │   └── common/               # Componentes compartidos
│   ├── nutri/                    # Componentes nutricionales
│   │   ├── MealCard.tsx
│   │   ├── AddFoodModal.tsx
│   │   ├── AddFoodModalAI.tsx    # 🤖 Con IA
│   │   └── FoodScanner.tsx       # Scanner código de barras
│   ├── stats/                    # Componentes de estadísticas
│   ├── services/                 # Servicios e integraciones
│   │   ├── GeminiFoodAnalyzer.ts # 🧠 Análisis con IA
│   │   └── FoodAnalysisService.ts
│   └── ui/                       # Componentes UI base
│
├── constants/                    # Constantes (colores, tokens)
├── hooks/                        # Custom hooks
├── assets/                       # Imágenes, iconos, fuentes
├── docs/                         # 📚 Documentación completa
└── app.json                      # Configuración Expo
```

---

## 🛠️ Scripts Disponibles

```bash
# Desarrollo
pnpm start                # Iniciar Metro bundler
pnpm android              # Abrir en Android
pnpm ios                  # Abrir en iOS
pnpm web                  # Abrir en navegador

# Calidad de código
pnpm lint                 # Ejecutar ESLint
pnpm lint --fix           # Auto-fix de errores

# Utilidades
pnpm reset-project        # Regenerar proyecto limpio
```

---

## ⚙️ Configuración

### Variables de Entorno

Crea un archivo `.env` en la raíz del proyecto:

```bash
# API Keys
GEMINI_API_KEY=tu_clave_aqui

# Opcional: Backend API
API_BASE_URL=https://api.alzan.com
```

> [!CAUTION]
> **Seguridad Importante**
> 
> Actualmente la API key de Gemini está hardcodeada en el código.  
> Antes de desplegar a producción, **mueve todas las claves a variables de entorno**.  
> Ver [SERVICES.md → Seguridad de API Keys](./docs/SERVICES.md#seguridad-de-api-keys)

---

### Configuración de `app.json`

Para personalizar la app (nombre, iconos, bundle IDs), edita [`app.json`](./app.json):

```json
{
  "expo": {
    "name": "Alzan",
    "slug": "alzan",
    "version": "1.0.0",
    "ios": {
      "bundleIdentifier": "com.tuempresa.alzan"
    },
    "android": {
      "package": "com.tuempresa.alzan"
    }
  }
}
```

---

##  Características Destacadas

### 🤖 Análisis de Comida con IA

```typescript
import { GeminiFoodAnalyzer } from '@/components/services/GeminiFoodAnalyzer';

// Analizar imagen
const result = await GeminiFoodAnalyzer.analyzeFood(imageUri);

console.log(result);
// {
//   detectedFood: "Ensalada César con pollo",
//   estimatedQuantity: "350g",
//   nutrition: {
//     calories: 420,
//     protein: 35,
//     carbs: 20,
//     fats: 18
//   },
//   confidence: 85,
//   ingredients: ["lechuga", "pollo", "queso parmesano"],
//   category: "real"
// }
```

Ver documentación completa en [SERVICES.md](./docs/SERVICES.md#servicio-de-análisis-con-ia)

---

### 🏋️ Sesión de Gimnasio Activa

```tsx
<ActiveWorkoutSession
  exercises={[
    { name: 'Sentadilla', sets: 4, reps: 10, weight: 80, restTime: 90 },
    { name: 'Press Banca', sets: 3, reps: 8, weight: 60, restTime: 120 }
  ]}
  onComplete={(summary) => saveWorkout(summary)}
  onCancel={() => router.back()}
/>
```

Incluye:
- Timer automático de descanso
- Registro de peso por serie
- Vibración al completar sets
- Registro de RPE post-entreno

Ver documentación en [COMPONENTS.md](./docs/COMPONENTS.md#activeworkoutsessiontsx)

---

## 🤝 Contribuir

¡Las contribuciones son bienvenidas! 🎉

### Primeros Pasos

1. **Fork** el repositorio
2. **Crea** una rama: `git checkout -b feature/mi-feature`
3. **Commit** cambios: `git commit -m "feat(sport): add new exercise type"`
4. **Push**: `git push origin feature/mi-feature`
5. **Abre** una Pull Request

### Guías

- Lee [CONTRIBUTING.md](./docs/CONTRIBUTING.md) para convenciones de código
- Usa **Conventional Commits**: `tipo(ámbito): descripción`
- Ejecuta `pnpm lint` antes de hacer commit
- Añade tests si es posible (pendiente de configurar)

---

## 🗺️ Roadmap

### ✅ Versión 1.0 (Actual)

- [x] Dashboard con resumen diario
- [x] Planificador deportivo multi-deporte
- [x] Seguimiento nutricional con IA
- [x] Panel de estadísticas
- [x] Gradientes y animaciones premium

### 🚧 Versión 1.1 (Próxima)

- [ ] Backend con Supabase
- [ ] Autenticación de usuarios
- [ ] Sincronización entre dispositivos
- [ ] Push notifications para recordatorios
- [ ] Integración con Google Fit / Apple HealthKit
- [ ] Tests automatizados (Jest + Testing Library)

### 🔮 Versión 2.0 (Futuro)

- [ ] Social: Compartir entrenamientos
- [ ] Planificador semanal de comidas
- [ ] Recetario con valores nutricionales
- [ ] Modo entrenador (asignar planes a clientes)
- [ ] Exportación PDF de reportes
- [ ] Integración con wearables (Apple Watch, Garmin)

---

## 📜 Licencia

Este proyecto es **privado** y no tiene licencia pública.  
Todos los derechos reservados © 2025.

---

## 📞 Contacto y Soporte

- **Issues**: [GitHub Issues](https://github.com/tu-usuario/alzan/issues)
- **Discusiones**: [GitHub Discussions](https://github.com/tu-usuario/alzan/discussions)
- **Email**: alzan-dev@example.com

---

## 🙏 Agradecimientos

Construido con:
- [Expo](https://expo.dev) - Framework y herramientas
- [React Native](https://reactnative.dev) - UI multiplataforma
- [Google Gemini](https://ai.google.dev) - Análisis con IA
- [OpenFoodFacts](https://world.openfoodfacts.org) - Base de datos de productos
- [React Native Paper](https://reactnativepaper.com) - Componentes Material Design

---

<div align="center">

**Hecho con ❤️ para mejorar tu salud y fitness**

[⬆ Volver arriba](#alzan--plataforma-de-fitness-y-nutrición-)

</div>
