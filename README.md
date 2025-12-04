# Alzan · Plataforma de Fitness y Nutrición

Aplicación móvil construida con Expo Router que unifica planificación deportiva, seguimiento nutricional y paneles de progreso en una sola experiencia. Incluye flujos para crear entrenamientos multi‑deporte, registrar comidas, analizar alimentos con IA (Gemini) y revisar métricas diarias/semanales con una interfaz basada en gradientes.

## Funcionalidades clave
- **Inicio motivacional**: tarjetas de calorías, pasos y hábitos semanales con acciones rápidas hacia entrenamiento y nutrición (`app/(tabs)/index.tsx`).
- **Planificador deportivo**: organiza un plan semanal por deporte, sesiones tipo gimnasio/carrera/ciclismo/natación, timers de descanso y registro de intensidad post‑entreno (`app/(tabs)/work.tsx`, `components/sport/**`).
- **Registro nutricional**: agenda de comidas, macros por sección, control de hidratación y modales para añadir alimentos o crear nuevos horarios (`app/(tabs)/nutrition.tsx`, `components/nutri/**`).
- **Panel de métricas**: componentes reutilizables para progreso, récords y consistencia disponibles en `components/stats/**`.
- **Análisis con IA**: `GeminiFoodAnalyzer` consume la API de Google Gemini para estimar nutrientes a partir de fotografías (`components/services/GeminiFoodAnalyzer.ts`).

## Stack técnico
- **Framework**: Expo 53 + React Native 0.79 + React 19.
- **Routing**: `expo-router` con navegación basada en archivos.
- **UI/UX**: `react-native-paper`, `@expo/vector-icons`, gradientes vía `expo-linear-gradient`.
- **Estado/UI**: hooks de React con tipado estricto (TypeScript 5.8, `tsconfig` hereda de Expo).
- **Utilidades**: módulos compartidos para deportes, timers y servicios de IA.

## Requisitos
- Node.js 18+ (LTS recomendado) y npm 9+.
- Expo CLI (`npm install -g expo-cli`) para comandos locales, emuladores o Expo Go.
- iOS Simulator (Xcode) o Android Studio si deseas ejecutar en simuladores/emuladores nativos.

## Puesta en marcha
```bash
git clone <repo-url>
cd alzan
npm install
npm run start      # abre Expo en modo interactivo
```

Expo CLI mostrará un QR y atajos (`i`, `a`, `w`) para iOS, Android o Web. Usa `npm run android` / `npm run ios` / `npm run web` si prefieres comandos directos.

## Scripts disponibles
- `npm run start`: lanza Metro + Expo Router.
- `npm run android` / `npm run ios` / `npm run web`: abre el proyecto en cada plataforma.
- `npm run lint`: ejecuta ESLint con la configuración `eslint-config-expo`.
- `npm run reset-project`: script utilitario que regenera un esqueleto limpio dentro de `app/` (ver `scripts/reset-project.js`).

## Estructura recomendada
```
app/                # Rutas Expo Router (tabs: inicio, work, nutrition, stats)
components/
  sport/            # UI y lógica de sesiones deportivas (gym, timers, modales)
  nutri/            # Modales y tarjetas nutricionales
  stats/            # Widgets de métricas y consistencia
  services/         # Integraciones externas (ej. Gemini)
assets/             # Iconos, fuentes y data mock
constants/          # Colores, tipografías y tokens compartidos
hooks/              # Hooks de theming/esquema de color
```

## Configuración y variables sensibles
- El servicio de Gemini usa una API key embebida en `components/services/GeminiFoodAnalyzer.ts`. Sustituye el valor por una variable segura (por ejemplo, usando `.env` + `expo-constants` o `app.config.js`) antes de publicar.
- Si añades más servicios externos, mantén las claves fuera del repositorio (`.env*.local` ya está ignorado en `.gitignore`).

## Próximos pasos sugeridos
- Añadir pruebas automatizadas (unitarias/UI) para módulos críticos.
- Externalizar datos mock hacia un backend o storage local.
- Implementar sincronización real para planes de entrenamiento y diario nutricional.

---

¿Necesitas más documentación (diagramas, guías de contribución, pruebas)? Abre un issue o continúa ampliando este README conforme evolucione el proyecto. 💪🥗

