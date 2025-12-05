# Guía de Despliegue - Al zan

## Índice
1. [Preparación para Producción](#preparación-para-producción)
2. [Build para iOS](#build-para-ios)
3. [Build para Android](#build-para-android)
4. [Build para Web](#build-para-web)
5. [EAS Build (Expo Application Services)](#eas-build)
6. [Distribución](#distribución)
7. [CI/CD](#cicd)
8. [Troubleshooting](#troubleshooting)

---

## Preparación para Producción

### 1. Verificar Configuración

**Checklist previo al deploy**:

- [ ] Todas las API keys movidas a variables de entorno
- [ ] `app.json` configurado correctamente
- [ ] Assets (iconos, splash screens) en alta resolución
- [ ] Versión incrementada en `app.json`
- [ ] Linter sin errores: `pnpm lint`
- [ ] Código testeado en iOS y Android
- [ ] Privacy Policy y Terms of Service preparados

---

### 2. Configurar app.json

```json
{
  "expo": {
    "name": "Alzan",
    "slug": "alzan",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/images/Appicon.png",
    "scheme": "alzan",
    "userInterfaceStyle": "automatic",
    "newArchEnabled": true,
    
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.tuempresa.alzan",
      "buildNumber": "1"
    },
    
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/images/Appicon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.tuempresa.alzan",
      "versionCode": 1,
      "permissions": [
        "CAMERA",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE"
      ]
    },
    
    "web": {
      "bundler": "metro",
      "output": "static",
      "favicon": "./assets/images/favicon.png"
    },
    
    "plugins": [
      "expo-router",
      [
        "expo-splash-screen",
        {
          "image": "./assets/images/splash-icon.png",
          "imageWidth": 200,
          "resizeMode": "contain",
          "backgroundColor": "#ffffff"
        }
      ],
      "expo-localization"
    ],
    
    "experiments": {
      "typedRoutes": true
    },
    
    "extra": {
      "eas": {
        "projectId": "tu-project-id-eas"
      }
    }
  }
}
```

---

### 3. Variables de Entorno

**Crear archivo** `eas.json`:
```json
{
  "cli": {
    "version": ">= 5.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "env": {
        "APP_ENV": "development"
      }
    },
    "preview": {
      "distribution": "internal",
      "env": {
        "APP_ENV": "preview"
      }
    },
    "production": {
      "env": {
        "APP_ENV": "production"
      }
    }
  },
  "submit": {
    "production": {}
  }
}
```

**Crear** `.env.production`:
```bash
GEMINI_API_KEY=tu_clave_produccion
API_BASE_URL=https://api.alzan.com
```

---

## Build para iOS

### Método 1: EAS Build (Recomendado)

**Ventajas**:
- Build en la nube (no necesitas Mac)
- Fácil distribución
- Integración con App Store Connect

**Pasos**:

**1. Instalar EAS CLI**
```bash
npm install -g eas-cli
```

**2. Login en Expo**
```bash
eas login
```

**3. Configurar proyecto**
```bash
eas build:configure
```

**4. Build para iOS**
```bash
# Build de producción
eas build --platform ios --profile production

# Build de preview (TestFlight)
eas build --platform ios --profile preview
```

**5. Esperar a que complete**
Build tarda ~15-30 minutos. Recibirás un enlace para descargar el `.ipa`.

---

### Método 2: Build Local (Requiere Mac)

**Requisitos**:
- macOS con Xcode instalado
- Apple Developer Account

**Pasos**:

**1. Pre-build**
```bash
npx expo prebuild --platform ios
```

Esto genera la carpeta `/ios` con archivos nativos.

**2. Abrir en Xcode**
```bash
open ios/Alzan.xcworkspace
```

**3. Configurar Signing**
- Selecciona tu equipo de desarrollo
- Configura Bundle Identifier
- Habilita capabilities necesarias (Camera, etc.)

**4. Build**
- Product → Archive
- Esperar a que complete
- Distribuir a App Store o TestFlight

---

### Configuración Adicional iOS

**Permisos en** `ios/Alzan/Info.plist`:
```xml
<key>NSCameraUsageDescription</key>
<string>Necesitamos acceso a la cámara para analizar alimentos</string>

<key>NSPhotoLibraryUsageDescription</key>
<string>Necesitamos acceso a fotos para seleccionar imágenes de comida</string>
```

---

## Build para Android

### Método 1: EAS Build (Recomendado)

**1. Build para Android**
```bash
# Build de producción (AAB para Play Store)
eas build --platform android --profile production

# Build de preview (APK para testing)
eas build --platform android --profile preview
```

**2. Descargar archivo**
- `.aab` (Android App Bundle) para Google Play
- `.apk` para distribución directa

---

### Método 2: Build Local

**Requisitos**:
- Android Studio instalado
- JDK 17+

**Pasos**:

**1. Pre-build**
```bash
npx expo prebuild --platform android
```

**2. Build AAB**
```bash
cd android
./gradlew bundleRelease
```

**3. Archivo generado en**:
```
android/app/build/outputs/bundle/release/app-release.aab
```

---

### Generar Keystore (Primera vez)

**Android requiere firmar apps con keystore**:

```bash
keytool -genkeypair -v \
  -storetype PKCS12 \
  -keystore alzan.keystore \
  -alias alzan-key \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

**Guardar credenciales de manera segura**:
- Keystore password
- Key alias
- Key password

> [!CAUTION]
> **Nunca subas el keystore a Git**
> 
> Si pierdes el keystore, no podrás actualizar la app en Play Store.  
> Guárdalo en un gestor de contraseñas o vault seguro.

---

**Configurar en** `android/app/build.gradle`:
```groovy
android {
    signingConfigs {
        release {
            storeFile file('../../alzan.keystore')
            storePassword System.getenv("KEYSTORE_PASSWORD")
            keyAlias 'alzan-key'
            keyPassword System.getenv("KEY_PASSWORD")
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

---

## Build para Web

### Build Estático

```bash
npx expo export:web
```

Genera archivos estáticos en `/web-build/`.

---

### Deploy a Vercel

**1. Instalar Vercel CLI**
```bash
npm install -g vercel
```

**2. Deploy**
```bash
vercel --prod
```

**3. Configurar dominio personalizado**
En dashboard de Vercel: Settings → Domains

---

### Deploy a Netlify

**1. Build**
```bash
npx expo export:web
```

**2. Crear** `netlify.toml`:
```toml
[build]
  command = "npx expo export:web"
  publish = "web-build"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

**3. Deploy**
```bash
npx netlify-cli deploy --prod
```

---

### Deploy a GitHub Pages

**1. Instalar gh-pages**
```bash
pnpm add -D gh-pages
```

**2. Añadir script en** `package.json`:
```json
{
  "scripts": {
    "deploy:web": "expo export:web && gh-pages -d web-build"
  }
}
```

**3. Deploy**
```bash
pnpm deploy:web
```

---

## EAS Build

### Configuración Completa

**eas.json**:
```json
{
  "cli": {
    "version": ">= 5.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "simulator": true
      }
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": false
      },
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "ios": {
        "simulator": false
      },
      "android": {
        "buildType": "app-bundle"
      }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "tu-email@example.com",
        "ascAppId": "1234567890",
        "appleTeamId": "ABCDE12345"
      },
      "android": {
        "serviceAccountKeyPath": "./service-account.json"
      }
    }
  }
}
```

---

### Secrets en EAS

**Añadir secrets para builds**:
```bash
eas secret:create --scope project --name GEMINI_API_KEY --value "tu_clave"
```

**Uso en código**:
```typescript
import Constants from 'expo-constants';

const GEMINI_API_KEY = Constants.expoConfig?.extra?.GEMINI_API_KEY;
```

---

### Build Automático

**Configurar en** `app.json`:
```json
{
  "expo": {
    "extra": {
      "eas": {
        "projectId": "tu-project-id"
      },
      "GEMINI_API_KEY": process.env.GEMINI_API_KEY
    }
  }
}
```

---

## Distribución

### App Store (iOS)

**Requisitos**:
- Apple Developer Account ($99/año)
- App Store Connect configurado

**Pasos**:

**1. Crear app en App Store Connect**
- Login: https://appstoreconnect.apple.com
- My Apps → + → New App
- Rellenar información (nombre, descripción, screenshots)

**2. Subir build**
```bash
eas submit --platform ios
```

O manualmente desde Xcode: Organizer → Upload to App Store

**3. Información requerida**:
- **Screenshots**: 6.5", 6.7" (iPhone), 12.9" (iPad)
- **App Privacy**: Qué datos recopilas
- **Age Rating**: Clasificación por edad
- **Keywords**: Para búsqueda
- **Description**: Máximo 4000 caracteres
- **Promotional Text**: Máximo 170 caracteres

**4. Enviar a revisión**
- Review tarda 1-3 días
- Posibles rechazos (cámara sin justificación, contenido, etc.)

---

### Google Play (Android)

**Requisitos**:
- Google Play Developer Account ($25 pago único)
- Política de privacidad publicada

**Pasos**:

**1. Crear app en Play Console**
- Login: https://play.google.com/console
- Create app
- Rellenar cuestionario

**2. Subir build**
```bash
eas submit --platform android
```

O manual: Production → Create new release → Upload AAB

**3. Información requerida**:
- **Screenshots**: Phone, 7" tablet, 10" tablet
- **Feature Graphic**: 1024x500px
- **App Icon**: 512x512px
- **Store Listing**: Descripción corta (80 chars) y larga (4000 chars)
- **Content Rating**: Cuestionario IARC
- **Privacy Policy**: URL pública

**4. Publicar**
- Revisión tarda 1-2 días
- Rollout gradual recomendado (10% → 50% → 100%)

---

### TestFlight (iOS Beta)

**Distribución beta para testers**:

```bash
eas build --platform ios --profile preview
eas submit --platform ios --profile preview
```

**Invitar testers**:
- App Store Connect → TestFlight
- External Testing → Add Testers
- Enviar invitaciones por email

---

### Internal Testing (Android)

```bash
eas build --platform android --profile preview
```

Distribuir APK directamente o usar Play Console Internal Testing.

---

## CI/CD

### GitHub Actions

**Crear** `.github/workflows/eas-build.yml`:

```yaml
name: EAS Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 10
          
      - name: Install dependencies
        run: pnpm install
        
      - name: Lint
        run: pnpm lint
        
      - name: Setup EAS
        uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
          
      - name: Build iOS
        run: eas build --platform ios --non-interactive --no-wait
        
      - name: Build Android
        run: eas build --platform android --non-interactive --no-wait
```

**Añadir secret**:
- GitHub repo → Settings → Secrets → New repository secret
- Name: `EXPO_TOKEN`
- Value: Token de Expo (obtener en https://expo.dev/settings/access-tokens)

---

### Automatizar Submit a Stores

**Crear** `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Stores

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup EAS
        uses: expo/expo-github-action@v8
        with:
          token: ${{ secrets.EXPO_TOKEN }}
          
      - name: Submit to App Store
        run: eas submit --platform ios --latest
        env:
          EXPO_APPLE_ID: ${{ secrets.EXPO_APPLE_ID }}
          EXPO_APPLE_APP_SPECIFIC_PASSWORD: ${{ secrets.EXPO_APPLE_APP_PASSWORD }}
          
      - name: Submit to Play Store
        run: eas submit --platform android --latest
```

---

## Troubleshooting

### Error: "Build failed"

**Solución**:
1. Ver logs completos: `eas build:list`
2. Verificar dependencias nativas
3. Limpiar caché: `pnpm clean` o `expo start -c`

---

### Error: "Keystore not found"

**Android**:
Asegúrate de que `alzan.keystore` existe y variables de entorno están configuradas:
```bash
export KEYSTORE_PASSWORD="tu_password"
export KEY_PASSWORD="tu_key_password"
```

---

### Error: "Provisioning profile doesn't match"

**iOS**:
1. Verificar Bundle ID en Xcode
2. Regenerar provisioning profile en Apple Developer
3. Sincronizar: Xcode → Preferences → Accounts → Download Manual Profiles

---

### App rechazada por Apple

**Razones comunes**:
- Falta justificación de permisos de cámara
- Crash al abrir
- UI incompleta o broken
- Contenido inapropiado

**Solución**:
1. Leer feedback de Apple Review Team
2. Corregir issues
3. Responder en Resolution Center
4. Reenviar para revisión

---

### APK muy grande

**Android**:
1. Usar AAB en lugar de APK (Play Store lo optimiza)
2. Habilitar ProGuard/R8 para minificación
3. Remover assets innecesarios

```gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
        }
    }
}
```

---

## Checklist Final de Deploy

### Antes de Lanzar

- [ ] Versión incrementada (`version` y `buildNumber`/`versionCode`)
- [ ] API keys en variables de entorno
- [ ] Assets en alta resolución (1024x1024 icon mínimo)
- [ ] Screenshots preparados (diferentes tamaños)
- [ ] Privacy Policy publicada
- [ ] Terms of Service publicados
- [ ] App testeada exhaustivamente
- [ ] Performance optimizada
- [ ] Logs de debug removidos
- [ ] Analytics configurado (opcional: Firebase, Mixpanel)
- [ ] Crash reporting configurado (opcional: Sentry, Bugsnag)

---

### Post-Lanzamiento

- [ ] Monitorear reviews/ratings
- [ ] Revisar crash reports
- [ ] Analizar métricas de uso
- [ ] Preparar hotfix si es necesario
- [ ] Responder a feedback de usuarios
- [ ] Planificar siguiente versión

---

## Recursos Útiles

- [Expo EAS Docs](https://docs.expo.dev/eas/)
- [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [Google Play Policies](https://play.google.com/about/developer-content-policy/)
- [Expo Forums](https://forums.expo.dev/)

---

¡Éxito con el lanzamiento de Alzan! 🚀
