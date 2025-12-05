# Guía de Servicios e Integraciones - Alzan

## Índice
1. [Servicio de Análisis con IA](#servicio-de-análisis-con-ia)
2. [Integración con OpenFoodFacts](#integración-con-openfoodfacts)
3. [Almacenamiento Local](#almacenamiento-local)
4. [Servicios de Cámara y Medios](#servicios-de-cámara-y-medios)
5. [Notificaciones y Feedback](#notificaciones-y-feedback)
6. [Exportación de Datos](#exportación-de-datos)
7. [Futuras Integraciones](#futuras-integraciones)

---

## Servicio de Análisis con IA

### GeminiFoodAnalyzer.ts

**Ubicación**: [`components/services/GeminiFoodAnalyzer.ts`](file:///Users/josep/Projectos/alzan-old/components/services/GeminiFoodAnalyzer.ts)

#### Descripción General
Servicio que utiliza **Google Gemini 2.0 Flash** (Vision API) para analizar imágenes de comida y extraer información nutricional automáticamente.

---

#### Configuración

```typescript
const GEMINI_API_KEY = 'AIzaSyAq-aQRkJeyS_1x5gwFLSsSW7yUJUenENQ'; // ⚠️ Hardcodeada
const GEMINI_API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent';
```

> [!CAUTION]
> **Problema de seguridad crítico**
> 
> La API key está expuesta directamente en el código fuente. Esto permite:
> - Uso no autorizado de tu cuota de API
> - Posibles cargos inesperados
> - Riesgo de revocación por Google
>
> **Solución recomendada**: Ver sección [Seguridad de API Keys](#seguridad-de-api-keys)

---

#### Interfaces y Tipos

```typescript
// Datos nutricionales devueltos por la IA
interface NutritionData {
  calories: number;      // kcal
  protein: number;       // gramos
  carbs: number;         // gramos
  fats: number;          // gramos
  fiber?: number;        // gramos (opcional)
  sugar?: number;        // gramos (opcional)
}

// Resultado completo del análisis
interface FoodAnalysisResult {
  detectedFood: string;           // "Ensalada César con pollo"
  estimatedQuantity: string;      // "300g"
  nutrition: NutritionData;
  confidence: number;             // 0-100
  ingredients: string[];          // ["lechuga", "pollo", "queso parmesano"]
  category: 'real' | 'good' | 'ultra'; // Clasificación nutricional
}

// Respuesta raw de Gemini
interface GeminiResponse {
  candidates: {
    content: {
      parts: { text: string }[];
    };
  }[];
}
```

---

#### Métodos Principales

##### 1. `analyzeFood(imageUri: string)`

**Propósito**: Analizar una imagen y obtener información nutricional completa

**Parámetros**:
- `imageUri` (string): URI de la imagen (file://, content://, etc.)

**Retorna**: `Promise<FoodAnalysisResult>`

**Flujo interno**:
```mermaid
graph TD
    A[analyzeFood] --> B[convertImageToBase64]
    B --> C[createFoodAnalysisPrompt]
    C --> D[Fetch Gemini API]
    D --> E[parseGeminiResponse]
    E --> F[validateAnalysisResult]
    F --> G[Return FoodAnalysisResult]
    
    D -->|Error| H[Throw Error]
    E -->|Invalid JSON| H
    F -->|Invalid Data| H
```

**Ejemplo de uso**:
```typescript
import { GeminiFoodAnalyzer } from '@/components/services/GeminiFoodAnalyzer';

// En un componente
const handleAnalyzeImage = async (imageUri: string) => {
  try {
    const result = await GeminiFoodAnalyzer.analyzeFood(imageUri);
    
    console.log('Comida detectada:', result.detectedFood);
    console.log('Cantidad:', result.estimatedQuantity);
    console.log('Calorías:', result.nutrition.calories);
    console.log('Confianza:', result.confidence + '%');
    
    // Usar resultado para pre-rellenar formulario
    setFoodData({
      name: result.detectedFood,
      quantity: parseFloat(result.estimatedQuantity),
      macros: result.nutrition,
      category: result.category
    });
    
  } catch (error) {
    console.error('Error al analizar:', error);
    Alert.alert('Error', 'No se pudo analizar la imagen');
  }
};
```

---

##### 2. `convertImageToBase64(imageUri: string)`

**Propósito**: Convertir imagen a formato base64 para enviar a API

**Parámetros**:
- `imageUri` (string): URI de la imagen

**Retorna**: `Promise<string>` (imagen en base64 sin prefijo `data:image/...`)

**Implementación**:
```typescript
private async convertImageToBase64(imageUri: string): Promise<string> {
  return new Promise((resolve, reject) => {
    fetch(imageUri)
      .then(res => res.blob())
      .then(blob => {
        const reader = new FileReader();
        
        reader.onloadend = () => {
          const base64 = reader.result as string;
          // Remover prefijo "data:image/jpeg;base64,"
          const cleanBase64 = base64.split(',')[1];
          resolve(cleanBase64);
        };
        
        reader.onerror = reject;
        reader.readAsDataURL(blob);
      })
      .catch(reject);
  });
}
```

---

##### 3. `createFoodAnalysisPrompt()`

**Propósito**: Generar prompt optimizado para análisis nutricional

**Retorna**: `string` (prompt en español)

**Prompt completo**:
```
Analiza esta imagen de comida y proporciona la siguiente información en formato JSON:

{
  "detectedFood": "Nombre del plato o alimento principal",
  "estimatedQuantity": "Cantidad estimada (ej: 250g, 1 porción)",
  "nutrition": {
    "calories": número (kcal totales),
    "protein": número (gramos),
    "carbs": número (gramos),
    "fats": número (gramos),
    "fiber": número (gramos, opcional),
    "sugar": número (gramos, opcional)
  },
  "confidence": número entre 0-100 (nivel de confianza del análisis),
  "ingredients": ["ingrediente1", "ingrediente2", ...],
  "category": "real" | "good" | "ultra"
}

IMPORTANTE:
1. Si la imagen no contiene comida, devuelve confidence: 0
2. category: "real" = comida sin procesar, "good" = ligeramente procesada, "ultra" = ultraprocesada
3. Los valores nutricionales deben ser para la cantidad total visible en la imagen
4. Devuelve SOLO el JSON, sin texto adicional antes o después
5. Asegúrate de que el JSON sea válido y parseable
```

**Estrategias de Prompt Engineering aplicadas**:
- ✅ Instrucciones claras y numeradas
- ✅ Ejemplo de estructura JSON esperada
- ✅ Advertencias sobre casos edge (no hay comida)
- ✅ Especificación de formato de respuesta
- ✅ Definición de categorías con criterios

---

##### 4. `parseGeminiResponse(data: GeminiResponse)`

**Propósito**: Extraer y parsear JSON de la respuesta de Gemini

**Lógica de parsing**:
```typescript
private parseGeminiResponse(data: GeminiResponse): FoodAnalysisResult {
  // Intentar extraer texto de diferentes estructuras de respuesta
  let textContent = '';
  
  if (data.candidates?.[0]?.content?.parts?.[0]?.text) {
    textContent = data.candidates[0].content.parts[0].text;
  } else if (data.content?.parts?.[0]?.text) {
    textContent = data.content.parts[0].text;
  } else if (data.parts?.[0]?.text) {
    textContent = data.parts[0].text;
  } else if (data.text) {
    textContent = data.text;
  }
  
  // Limpiar texto (remover markdown, espacios, etc.)
  const cleanText = textContent
    .replace(/```json/g, '')
    .replace(/```/g, '')
    .trim();
  
  // Parsear JSON
  const result = JSON.parse(cleanText);
  
  // Validar estructura
  this.validateAnalysisResult(result);
  
  return result;
}
```

---

##### 5. `validateAnalysisResult(result: any)`

**Propósito**: Validar que la respuesta tenga la estructura correcta

**Validaciones**:
```typescript
private validateAnalysisResult(result: any): void {
  // Campos requeridos
  if (!result.detectedFood || typeof result.detectedFood !== 'string') {
    throw new Error('Campo "detectedFood" inválido');
  }
  
  if (!result.estimatedQuantity || typeof result.estimatedQuantity !== 'string') {
    throw new Error('Campo "estimatedQuantity" inválido');
  }
  
  // Validar objeto nutrition
  if (!result.nutrition || typeof result.nutrition !== 'object') {
    throw new Error('Campo "nutrition" inválido');
  }
  
  const { calories, protein, carbs, fats } = result.nutrition;
  
  if (typeof calories !== 'number' || calories < 0) {
    throw new Error('Calorías inválidas');
  }
  
  if (typeof protein !== 'number' || protein < 0) {
    throw new Error('Proteínas inválidas');
  }
  
  if (typeof carbs !== 'number' || carbs < 0) {
    throw new Error('Carbohidratos inválidos');
  }
  
  if (typeof fats !== 'number' || fats < 0) {
    throw new Error('Grasas inválidas');
  }
  
  // Validar confidence
  if (typeof result.confidence !== 'number' || 
      result.confidence < 0 || 
      result.confidence > 100) {
    throw new Error('Confidence debe estar entre 0 y 100');
  }
  
  // Validar ingredients
  if (!Array.isArray(result.ingredients)) {
    throw new Error('Ingredients debe ser un array');
  }
  
  // Validar category
  const validCategories = ['real', 'good', 'ultra'];
  if (!validCategories.includes(result.category)) {
    throw new Error('Category debe ser real, good o ultra');
  }
}
```

---

##### 6. `testConnection()`

**Propósito**: Verificar conectividad con la API de Gemini

**Retorna**: `Promise<boolean>`

**Uso**:
```typescript
// Al iniciar la app
const isConnected = await GeminiFoodAnalyzer.testConnection();

if (!isConnected) {
  Alert.alert(
    'Error de conexión',
    'No se pudo conectar con el servicio de análisis de IA'
  );
}
```

---

#### Funciones Helper Exportadas

```typescript
// Análisis directo (wrapper)
export const analyzeFoodImage = async (imageUri: string) => {
  return GeminiFoodAnalyzer.analyzeFood(imageUri);
};

// Verificar si está configurada
export const isGeminiConfigured = (): boolean => {
  return GEMINI_API_KEY !== '';
};
```

---

#### Manejo de Errores

**Tipos de errores posibles**:

1. **Error de red**:
```typescript
// Sin conexión a Internet
catch (error) {
  if (error.message.includes('Network request failed')) {
    Alert.alert('Sin conexión', 'Verifica tu conexión a Internet');
  }
}
```

2. **Error de API (401, 403, 429)**:
```typescript
// Rate limit excedido
if (response.status === 429) {
  Alert.alert('Límite excedido', 'Demasiadas solicitudes, espera un momento');
}

// API key inválida
if (response.status === 401 || response.status === 403) {
  Alert.alert('Error de autenticación', 'API key inválida');
}
```

3. **Error de parsing JSON**:
```typescript
catch (error) {
  if (error instanceof SyntaxError) {
    console.error('Respuesta no es JSON válido:', textContent);
    Alert.alert('Error', 'La IA no devolvió un formato válido');
  }
}
```

4. **Imagen no contiene comida**:
```typescript
if (result.confidence < 10) {
  Alert.alert(
    'No se detectó comida',
    'Asegúrate de que la imagen contenga alimentos visibles'
  );
}
```

---

#### Optimizaciones y Mejoras

##### Caché de Resultados
```typescript
// Implementación sugerida
const resultCache = new Map<string, FoodAnalysisResult>();

const analyzeWithCache = async (imageUri: string) => {
  // Generar hash de la imagen
  const imageHash = await hashImage(imageUri);
  
  // Verificar caché
  if (resultCache.has(imageHash)) {
    console.log('Resultado recuperado de caché');
    return resultCache.get(imageHash)!;
  }
  
  // Analizar y guardar en caché
  const result = await GeminiFoodAnalyzer.analyzeFood(imageUri);
  resultCache.set(imageHash, result);
  
  return result;
};
```

##### Compresión de Imágenes
```typescript
import * as ImageManipulator from 'expo-image-manipulator';

const compressImage = async (uri: string) => {
  const result = await ImageManipulator.manipulateAsync(
    uri,
    [{ resize: { width: 1024 } }], // Redimensionar a max 1024px
    { compress: 0.7, format: ImageManipulator.SaveFormat.JPEG }
  );
  
  return result.uri;
};

// Usar antes de analizar
const compressedUri = await compressImage(imageUri);
const analysis = await GeminiFoodAnalyzer.analyzeFood(compressedUri);
```

##### Retry Logic
```typescript
const analyzeWithRetry = async (imageUri: string, maxRetries = 3) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await GeminiFoodAnalyzer.analyzeFood(imageUri);
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      console.log(`Intento ${attempt} falló, reintentando...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
    }
  }
};
```

---

#### Seguridad de API Keys

**Problema actual**:
```typescript
// ❌ API key expuesta en código
const GEMINI_API_KEY = 'AIzaSyAq-aQRkJeyS_1x5gwFLSsSW7yUJUenENQ';
```

**Solución recomendada**:

**Paso 1**: Crear archivo `.env`
```bash
# .env
GEMINI_API_KEY=AIzaSyAq-aQRkJeyS_1x5gwFLSsSW7yUJUenENQ
```

**Paso 2**: Añadir al `.gitignore`
```bash
# .gitignore
.env
.env.local
.env.*.local
```

**Paso 3**: Configurar en `app.config.js`
```javascript
// app.config.js
import 'dotenv/config';

export default {
  expo: {
    name: "AlZan",
    // ... otras configuraciones
    extra: {
      geminiApiKey: process.env.GEMINI_API_KEY,
    }
  }
};
```

**Paso 4**: Acceder en código
```typescript
import Constants from 'expo-constants';

const GEMINI_API_KEY = Constants.expoConfig?.extra?.geminiApiKey || '';

if (!GEMINI_API_KEY) {
  console.error('⚠️ GEMINI_API_KEY no configurada');
}
```

**Paso 5**: Instalar dependencias
```bash
npm install dotenv
```

---

#### Costos y Límites de Gemini API

**Modelo**: Gemini 2.0 Flash

**Pricing (a fecha dic 2025)**:
- Gratuito: 15 solicitudes/minuto
- Gratis hasta 1,500 solicitudes/día
- Imágenes cuentan como ~258 tokens aprox.

**Límites de rate**:
- 15 RPM (requests per minute)
- 1M TPM (tokens per minute)
- 1,500 RPD (requests per day)

**Cálculo de uso**:
```
Si un usuario analiza 10 fotos/día:
→ 10 solicitudes × 30 días = 300 solicitudes/mes
→ Gratis ✅

Si tienes 100 usuarios activos:
→ 100 × 10 = 1,000 solicitudes/día
→ Gratis ✅

Si tienes 200 usuarios activos:
→ 200 × 10 = 2,000 solicitudes/día
→ Excede límite gratuito ⚠️
```

**Implementar límite por usuario**:
```typescript
const userLimits = new Map<string, number>();

const checkUserLimit = (userId: string) => {
  const today = new Date().toDateString();
  const key = `${userId}-${today}`;
  const count = userLimits.get(key) || 0;
  
  if (count >= 10) {
    throw new Error('Límite diario alcanzado (10 análisis/día)');
  }
  
  userLimits.set(key, count + 1);
};
```

---

## Integración con OpenFoodFacts

### Descripción
Base de datos colaborativa de productos alimenticios con código de barras.

**Website**: https://world.openfoodfacts.org  
**API Docs**: https://wiki.openfoodfacts.org/API

---

### Implementación

**Archivo sugerido**: `components/services/OpenFoodFactsService.ts`

```typescript
// OpenFoodFactsService.ts
const API_BASE = 'https://world.openfoodfacts.org/api/v0';

interface ProductData {
  code: string; // Código de barras
  product: {
    product_name: string;
    brands: string;
    quantity: string;
    nutriments: {
      'energy-kcal_100g': number;
      proteins_100g: number;
      carbohydrates_100g: number;
      fat_100g: number;
      fiber_100g?: number;
      sugars_100g?: number;
    };
    nova_group: number; // 1-4 (procesamiento)
    nutriscore_grade?: string; // A-E
  };
}

export class OpenFoodFactsService {
  static async searchByBarcode(barcode: string): Promise<ProductData | null> {
    try {
      const response = await fetch(
        `${API_BASE}/product/${barcode}.json`,
        {
          headers: {
            'User-Agent': 'Alzan - Fitness App - Version 1.0'
          }
        }
      );
      
      const data = await response.json();
      
      if (data.status === 1) {
        return data;
      }
      
      return null; // Producto no encontrado
      
    } catch (error) {
      console.error('Error buscando producto:', error);
      throw error;
    }
  }
  
  static formatProductData(data: ProductData) {
    return {
      name: data.product.product_name,
      brand: data.product.brands,
      quantity: data.product.quantity,
      macros: {
        calories: data.product.nutriments['energy-kcal_100g'],
        protein: data.product.nutriments.proteins_100g,
        carbs: data.product.nutriments.carbohydrates_100g,
        fats: data.product.nutriments.fat_100g,
        fiber: data.product.nutriments.fiber_100g,
        sugar: data.product.nutriments.sugars_100g
      },
      category: this.classifyByNovaGroup(data.product.nova_group),
      nutriScore: data.product.nutriscore_grade?.toUpperCase()
    };
  }
  
  static classifyByNovaGroup(novaGroup: number): 'real' | 'good' | 'ultra' {
    if (novaGroup === 1) return 'real';       // Sin procesar
    if (novaGroup === 2 || novaGroup === 3) return 'good'; // Procesado
    return 'ultra';                            // Ultraprocesado
  }
}
```

---

### Uso en FoodScanner

```tsx
// FoodScanner.tsx
import { OpenFoodFactsService } from '@/components/services/OpenFoodFactsService';

const handleBarcodeScan = async (barcode: string) => {
  setScanning(false);
  setLoading(true);
  
  try {
    const productData = await OpenFoodFactsService.searchByBarcode(barcode);
    
    if (productData) {
      const formatted = OpenFoodFactsService.formatProductData(productData);
      
      // Pre-rellenar formulario
      setFoodData({
        name: `${formatted.brand} ${formatted.name}`,
        quantity: 100, // Base 100g
        macros: formatted.macros,
        category: formatted.category,
        nutriScore: formatted.nutriScore
      });
      
      Alert.alert('Producto encontrado', formatted.name);
    } else {
      Alert.alert(
        'Producto no encontrado',
        'Este código de barras no está en la base de datos'
      );
    }
    
  } catch (error) {
    Alert.alert('Error', 'No se pudo buscar el producto');
  } finally {
    setLoading(false);
  }
};
```

---

## Almacenamiento Local

### AsyncStorage (React Native)

**Instalación**:
```bash
npx expo install @react-native-async-storage/async-storage
```

**Uso básico**:
```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

// Guardar datos
await AsyncStorage.setItem('weeklyPlan', JSON.stringify(plan));

// Recuperar datos
const savedPlan = await AsyncStorage.getItem('weeklyPlan');
const plan = savedPlan ? JSON.parse(savedPlan) : null;

// Eliminar
await AsyncStorage.removeItem('weeklyPlan');

// Limpiar todo
await AsyncStorage.clear();
```

---

### Custom Hook para Persistencia

**Archivo**: `hooks/usePersistentState.ts`

```typescript
import { useEffect, useState } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export function usePersistentState<T>(
  key: string, 
  initialValue: T
): [T, (value: T) => void] {
  const [state, setState] = useState<T>(initialValue);
  const [loaded, setLoaded] = useState(false);
  
  // Cargar al montar
  useEffect(() => {
    AsyncStorage.getItem(key).then(value => {
      if (value !== null) {
        setState(JSON.parse(value));
      }
      setLoaded(true);
    });
  }, [key]);
  
  // Guardar al cambiar
  useEffect(() => {
    if (loaded) {
      AsyncStorage.setItem(key, JSON.stringify(state));
    }
  }, [key, state, loaded]);
  
  return [state, setState];
}

// Uso
const [meals, setMeals] = usePersistentState('daily_meals', []);
```

---

## Servicios de Cámara y Medios

### expo-camera

**Permisos**:
```typescript
import { Camera } from 'expo-camera';

const requestPermissions = async () => {
  const { status } = await Camera.requestCameraPermissionsAsync();
  
  if (status !== 'granted') {
    Alert.alert(
      'Permiso denegado',
      'Necesitamos acceso a la cámara para analizar alimentos',
      [
        { text: 'Cancelar', style: 'cancel' },
        { text: 'Configuración', onPress: () => Linking.openSettings() }
      ]
    );
    return false;
  }
  
  return true;
};
```

---

### expo-image-picker

**Seleccionar de galería**:
```typescript
import * as ImagePicker from 'expo-image-picker';

const pickImage = async () => {
  const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
  
  if (status !== 'granted') {
    Alert.alert('Permiso denegado');
    return;
  }
  
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    allowsEditing: true,
    aspect: [4, 3],
    quality: 0.8,
  });
  
  if (!result.canceled) {
    const imageUri = result.assets[0].uri;
    return imageUri;
  }
  
  return null;
};
```

**Tomar foto**:
```typescript
const takePhoto = async () => {
  const { status } = await ImagePicker.requestCameraPermissionsAsync();
  
  if (status !== 'granted') {
    Alert.alert('Permiso denegado');
    return;
  }
  
  const result = await ImagePicker.launchCameraAsync({
    allowsEditing: true,
    aspect: [4, 3],
    quality: 0.8,
  });
  
  if (!result.canceled) {
    return result.assets[0].uri;
  }
  
  return null;
};
```

---

## Notificaciones y Feedback

### expo-haptics (Vibración)

```typescript
import * as Haptics from 'expo-haptics';

// Al completar set
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);

// Al completar entreno
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);

// Error
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);

// Selección (UI)
Haptics.selectionAsync();
```

---

### Push Notifications (futuro)

**Instalación**:
```bash
npx expo install expo-notifications
```

**Configuración básica**:
```typescript
import * as Notifications from 'expo-notifications';

// Solicitar permisos
const { status } = await Notifications.requestPermissionsAsync();

// Programar notificación
await Notifications.scheduleNotificationAsync({
  content: {
    title: "¡Hora de entrenar! 💪",
    body: 'Tienes una sesión de gimnasio programada',
  },
  trigger: {
    hour: 18,
    minute: 0,
    repeats: true
  },
});
```

---

## Exportación de Datos

### Exportar a Excel (xlsx)

**Ya instalado**: `xlsx@0.18.5`

```typescript
import * as XLSX from 'xlsx';
import * as FileSystem from 'expo-file-system';
import * as Sharing from 'expo-sharing';

const exportToExcel = async (data: any[]) => {
  // Crear workbook
  const ws = XLSX.utils.json_to_sheet(data);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Datos");
  
  // Generar archivo
  const wbout = XLSX.write(wb, { type: 'base64', bookType: 'xlsx' });
  
  // Guardar en filesystem
  const uri = FileSystem.documentDirectory + 'alzan_export.xlsx';
  await FileSystem.writeAsStringAsync(uri, wbout, {
    encoding: FileSystem.EncodingType.Base64,
  });
  
  // Compartir
  await Sharing.shareAsync(uri);
};

// Uso
const mealsData = meals.map(meal => ({
  Fecha: meal.date,
  Comida: meal.name,
  Calorías: meal.totalMacros.calories,
  Proteínas: meal.totalMacros.protein,
  Carbohidratos: meal.totalMacros.carbs,
  Grasas: meal.totalMacros.fats
}));

await exportToExcel(mealsData);
```

---

## Futuras Integraciones

### Supabase (Backend)

**Características**:
- Base de datos PostgreSQL
- Autenticación
- Storage de imágenes
- Real-time subscriptions

**Instalación**:
```bash
npm install @supabase/supabase-js
```

**Configuración**:
```typescript
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  'https://tu-proyecto.supabase.co',
  'tu-anon-key'
);

// Insertar comida
const { data, error } = await supabase
  .from('meals')
  .insert({
    user_id: userId,
    name: 'Desayuno',
    foods: JSON.stringify(foods),
    total_calories: 450
  });

// Consultar entrenamientos
const { data: workouts } = await supabase
  .from('workouts')
  .select('*')
  .eq('user_id', userId)
  .gte('date', startDate)
  .lte('date', endDate);
```

---

### Google Fit / Apple HealthKit

**Sincronizar pasos, calorías, frecuencia cardíaca**

```bash
npm install react-native-health
```

```typescript
import AppleHealthKit from 'react-native-health';

// Solicitar permisos
AppleHealthKit.initHealthKit(permissions, (error) => {
  if (error) return;
  
  // Obtener pasos del día
  AppleHealthKit.getStepCount(options, (err, results) => {
    console.log('Pasos hoy:', results.value);
  });
});
```

---

### Stripe (Suscripciones Premium)

**Para monetización futura**

```bash
npm install @stripe/stripe-react-native
```

```typescript
import { useStripe } from '@stripe/stripe-react-native';

const { initPaymentSheet, presentPaymentSheet } = useStripe();

// Configurar payment sheet
await initPaymentSheet({
  merchantDisplayName: 'Alzan',
  customerId: user.stripeId,
  customerEphemeralKeySecret: ephemeralKey,
  paymentIntentClientSecret: paymentIntent,
});

// Mostrar
const { error } = await presentPaymentSheet();

if (!error) {
  Alert.alert('¡Gracias!', 'Suscripción activada');
}
```

---

## Diagrama de Servicios

```mermaid
graph TB
    A[App Components] --> B[GeminiFoodAnalyzer]
    A --> C[OpenFoodFactsService]
    A --> D[AsyncStorage]
    A --> E[ImagePicker]
    
    B --> F[Google Gemini API]
    C --> G[OpenFoodFacts API]
    D --> H[Local Storage]
    E --> I[Device Camera/Gallery]
    
    A --> J[Haptics]
    A --> K[Notifications]
    
    style B fill:#4285F4
    style C fill:#34A853
    style F fill:#EA4335
    style G fill:#FBBC04
```

---

## Checklist de Seguridad

- [x] Solicitar permisos de cámara antes de usar
- [x] Validar respuestas de APIs externas
- [ ] Mover API keys a variables de entorno
- [ ] Implementar rate limiting por usuario
- [ ] Encriptar datos sensibles con SecureStore
- [ ] Sanitizar inputs del usuario
- [ ] Implementar timeout en llamadas de red
- [ ] Logging de errores (Sentry/Bugsnag)

---

Para más información:
- [Arquitectura](./ARCHITECTURE.md)
- [Componentes](./COMPONENTS.md)
- [Contribución](./CONTRIBUTING.md)
