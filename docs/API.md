# API Reference - Alzan

## Índice
1. [Componentes Principales](#componentes-principales)
2. [Servicios](#servicios)
3. [Hooks Personalizados](#hooks-personalizados)
4. [Tipos y Interfaces](#tipos-y-interfaces)
5. [Utilidades](#utilidades)

---

## Componentes Principales

### Componentes Deportivos

#### `<ActiveWorkoutSession />`

Componente para gestionar sesiones de entrenamiento activas.

**Props**:
```typescript
interface ActiveWorkoutSessionProps {
  exercises: Exercise[];
  onComplete: (summary: WorkoutSummary) => void;
  onCancel: () => void;
}
```

**Uso**:
```tsx
<ActiveWorkoutSession
  exercises={workoutExercises}
  onComplete={handleComplete}
  onCancel={handleCancel}
/>
```

---

#### `<GymSessionModal />`

Modal para configurar sesiones de gimnasio.

**Props**:
```typescript
interface GymSessionModalProps {
  visible: boolean;
  onClose: () => void;
  onStart: (config: SessionConfig) => void;
  initialData?: SessionConfig;
}
```

---

### Componentes Nutricionales

#### `<MealCard />`

Tarjeta que muestra información de una comida.

**Props**:
```typescript
interface MealCardProps {
  meal: Meal;
  onEdit?: () => void;
  onDelete?: () => void;
  expanded?: boolean;
}
```

---

#### `<AddFoodModalAI />`

Modal con análisis de IA para añadir comida.

**Props**:
```typescript
interface AddFoodModalAIProps {
  visible: boolean;
  mealId: string;
  onClose: () => void;
  onSave: (food: Food) => void;
}
```

**Ejemplo**:
```tsx
<AddFoodModalAI
  visible={showModal}
  mealId="breakfast"
  onClose={() => setShowModal(false)}
  onSave={handleSaveFood}
/>
```

---

## Servicios

### GeminiFoodAnalyzer

Servicio para análisis de comida con IA.

#### `analyzeFood(imageUri: string): Promise<FoodAnalysisResult>`

Analiza una imagen y devuelve información nutricional.

**Parámetros**:
- `imageUri` (string): URI de la imagen a analizar

**Retorna**:
```typescript
interface FoodAnalysisResult {
  detectedFood: string;
  estimatedQuantity: string;
  nutrition: NutritionData;
  confidence: number;
  ingredients: string[];
  category: 'real' | 'good' | 'ultra';
}
```

**Ejemplo**:
```typescript
import { GeminiFoodAnalyzer } from '@/components/services/GeminiFoodAnalyzer';

const result = await GeminiFoodAnalyzer.analyzeFood(imageUri);
console.log(result.detectedFood); // "Ensalada mixta"
```

**Errores**:
- `Error`: Si la imagen no es válida
- `Error`: Si la API falla
- `Error`: Si la respuesta no es JSON válido

---

#### `testConnection(): Promise<boolean>`

Verifica conectividad con la API de Gemini.

**Retorna**: `true` si la conexión es exitosa, `false` en caso contrario.

**Ejemplo**:
```typescript
const isConnected = await GeminiFoodAnalyzer.testConnection();
if (!isConnected) {
  Alert.alert('Error', 'No se pudo conectar con el servicio de IA');
}
```

---

### OpenFoodFactsService

Servicio para buscar productos por código de barras.

#### `searchByBarcode(barcode: string): Promise<ProductData | null>`

Busca un producto en OpenFoodFacts.

**Parámetros**:
- `barcode` (string): Código de barras del producto

**Retorna**: Datos del producto o `null` si no se encuentra.

**Ejemplo**:
```typescript
import { OpenFoodFactsService } from '@/components/services/OpenFoodFactsService';

const product = await OpenFoodFactsService.searchByBarcode('8480000123456');
if (product) {
  console.log(product.product.product_name);
}
```

---

## Hooks Personalizados

### `useColorScheme()`

Hook para obtener el esquema de color del dispositivo.

**Retorna**: `'light' | 'dark'`

**Uso**:
```typescript
import { useColorScheme } from '@/hooks/useColorScheme';

const MyComponent = () => {
  const colorScheme = useColorScheme();
  const bgColor = colorScheme === 'dark' ? '#000' : '#FFF';
  
  return <View style={{ backgroundColor: bgColor }} />;
};
```

---

### `useThemeColor(colorName: string)`

Hook para obtener un color temático.

**Parámetros**:
- `colorName` (string): Nombre del color en la paleta

**Retorna**: Código de color hex (string)

**Uso**:
```typescript
import { useThemeColor } from '@/hooks/useThemeColor';

const MyComponent = () => {
  const primaryColor = useThemeColor('primary');
  
  return <Text style={{ color: primaryColor }}>Texto</Text>;
};
```

---

### `usePersistentState<T>(key: string, initialValue: T)`

Hook para estado persistente con AsyncStorage.

**Parámetros**:
- `key` (string): Clave para almacenamiento
- `initialValue` (T): Valor inicial

**Retorna**: `[T, (value: T) => void]` (similar a useState)

**Uso**:
```typescript
import { usePersistentState } from '@/hooks/usePersistentState';

const MyComponent = () => {
  const [meals, setMeals] = usePersistentState('daily_meals', []);
  
  const addMeal = (meal) => {
    setMeals([...meals, meal]);
  };
  
  return <MealList meals={meals} onAdd={addMeal} />;
};
```

---

## Tipos y Interfaces

### Tipos de Datos Deportivos

#### `Exercise`

```typescript
interface Exercise {
  id: string;
  name: string;
  sets: number;
  reps: number;
  weight?: number;
  restTime?: number; // segundos
}
```

#### `WorkoutSummary`

```typescript
interface WorkoutSummary {
  date: Date;
  sport: string;
  duration: number; // minutos
  exercises?: Exercise[];
  totalVolume?: number; // kg levantados
  distance?: number; // km
  averageHeartRate?: number;
  intensity?: IntensityData;
}
```

#### `IntensityData`

```typescript
interface IntensityData {
  rpe: number; // 1-10
  difficulty: 'easy' | 'moderate' | 'hard' | 'very_hard';
  notes?: string;
  musclesSore?: string[];
}
```

---

### Tipos de Datos Nutricionales

#### `Meal`

```typescript
interface Meal {
  id: string;
  name: string; // "Desayuno", "Almuerzo", etc.
  time: string; // "08:00"
  foods: Food[];
  totalMacros: Macros;
}
```

#### `Food`

```typescript
interface Food {
  id: string;
  name: string;
  quantity: number;
  unit: 'g' | 'ml' | 'unit' | 'portion';
  macros: Macros;
  category: 'real' | 'good' | 'ultra';
}
```

#### `Macros`

```typescript
interface Macros {
  calories: number;
  protein: number;
  carbs: number;
  fats: number;
  fiber?: number;
  sugar?: number;
}
```

---

### Tipos de Respuestas de IA

#### `FoodAnalysisResult`

```typescript
interface FoodAnalysisResult {
  detectedFood: string;
  estimatedQuantity: string;
  nutrition: NutritionData;
  confidence: number; // 0-100
  ingredients: string[];
  category: 'real' | 'good' | 'ultra';
}
```

#### `NutritionData`

```typescript
interface NutritionData {
  calories: number;
  protein: number;
  carbs: number;
  fats: number;
  fiber?: number;
  sugar?: number;
}
```

---

## Utilidades

### Funciones Helper

#### `calculateTotalMacros(foods: Food[]): Macros`

Calcula macros totales de un array de alimentos.

**Parámetros**:
- `foods` (Food[]): Array de alimentos

**Retorna**: Objeto `Macros` con totales

**Ejemplo**:
```typescript
const foods = [
  { macros: { calories: 100, protein: 10, carbs: 15, fats: 3 } },
  { macros: { calories: 200, protein: 15, carbs: 20, fats: 8 } }
];

const totals = calculateTotalMacros(foods);
// { calories: 300, protein: 25, carbs: 35, fats: 11 }
```

---

#### `formatTime(seconds: number): string`

Formatea segundos a formato MM:SS.

**Parámetros**:
- `seconds` (number): Segundos a formatear

**Retorna**: String en formato "MM:SS"

**Ejemplo**:
```typescript
formatTime(90);  // "01:30"
formatTime(125); // "02:05"
```

---

#### `calculatePace(distance: number, duration: number): string`

Calcula ritmo (pace) para running.

**Parámetros**:
- `distance` (number): Distancia en km
- `duration` (number): Duración en minutos

**Retorna**: Ritmo en formato "MM:SS min/km"

**Ejemplo**:
```typescript
calculatePace(5, 27.5); // "05:30 min/km"
calculatePace(10, 50);  // "05:00 min/km"
```

---

#### `classifyFood(novaGroup: number): 'real' | 'good' | 'ultra'`

Clasifica alimento según grupo NOVA.

**Parámetros**:
- `novaGroup` (number): Grupo NOVA (1-4)

**Retorna**: Categoría de alimento

**Ejemplo**:
```typescript
classifyFood(1); // "real" (sin procesar)
classifyFood(2); // "good" (procesado)
classifyFood(4); // "ultra" (ultraprocesado)
```

---

### Constantes

#### `COLORS`

Paleta de colores de la aplicación.

```typescript
export const COLORS = {
  primary: '#6C63FF',
  secondary: '#00D4AA',
  accent: '#FF6584',
  background: '#1A1A2E',
  surface: '#16213E',
  text: '#EAEAEA',
  textSecondary: '#B0B0C4',
  error: '#FF4757',
  success: '#00D4AA',
  warning: '#FFA502',
};
```

---

#### `GRADIENTS`

Gradientes predefinidos.

```typescript
export const GRADIENTS = {
  primary: ['#6C63FF', '#4B47CC'],
  success: ['#00D4AA', '#00A67E'],
  danger: ['#FF6584', '#D93654'],
  warm: ['#FF9A56', '#FF6584'],
  cool: ['#4FACFE', '#00F2FE'],
};
```

---

#### `SPORT_TYPES`

Tipos de deportes soportados.

```typescript
export const SPORT_TYPES = {
  GYM: 'gym',
  RUNNING: 'running',
  CYCLING: 'cycling',
  SWIMMING: 'swimming',
  OTHER: 'other',
} as const;
```

---

## Ejemplos Completos

### Ejemplo 1: Flujo Completo de Gimnasio

```tsx
import { useState } from 'react';
import { GymSessionModal } from '@/components/sport/gym/GymSessionModal';
import { ActiveWorkoutSession } from '@/components/sport/gym/ActiveWorkoutSession';
import { WorkoutIntensityModal } from '@/components/sport/gym/WorkoutIntensityModal';

export default function WorkoutScreen() {
  const [showConfig, setShowConfig] = useState(false);
  const [activeSession, setActiveSession] = useState(null);
  const [showIntensity, setShowIntensity] = useState(false);

  const handleStartWorkout = (config) => {
    setShowConfig(false);
    setActiveSession(config);
  };

  const handleCompleteWorkout = (summary) => {
    setActiveSession(null);
    setShowIntensity(true);
  };

  const handleIntensitySubmit = (intensity) => {
    saveWorkout({ ...workoutSummary, intensity });
    setShowIntensity(false);
  };

  return (
    <>
      <GymSessionModal
        visible={showConfig}
        onClose={() => setShowConfig(false)}
        onStart={handleStartWorkout}
      />
      
      {activeSession && (
        <ActiveWorkoutSession
          exercises={activeSession.exercises}
          onComplete={handleCompleteWorkout}
          onCancel={() => setActiveSession(null)}
        />
      )}
      
      <WorkoutIntensityModal
        visible={showIntensity}
        onSubmit={handleIntensitySubmit}
        onClose={() => setShowIntensity(false)}
      />
    </>
  );
}
```

---

### Ejemplo 2: Análisis de Comida con IA

```tsx
import { useState } from 'react';
import { Alert } from 'react-native';
import * as ImagePicker from 'expo-image-picker';
import { GeminiFoodAnalyzer } from '@/components/services/GeminiFoodAnalyzer';

export default function NutritionScreen() {
  const [analyzing, setAnalyzing] = useState(false);
  const [foodData, setFoodData] = useState(null);

  const handleAnalyze = async () => {
    // 1. Solicitar permiso
    const { status } = await ImagePicker.requestCameraPermissionsAsync();
    if (status !== 'granted') {
      Alert.alert('Permiso denegado');
      return;
    }

    // 2. Capturar imagen
    const result = await ImagePicker.launchCameraAsync({
      quality: 0.8,
      allowsEditing: true,
    });

    if (result.canceled) return;

    // 3. Analizar con IA
    setAnalyzing(true);
    try {
      const analysis = await GeminiFoodAnalyzer.analyzeFood(result.assets[0].uri);
      
      // 4. Pre-rellenar formulario
      setFoodData({
        name: analysis.detectedFood,
        quantity: parseFloat(analysis.estimatedQuantity),
        macros: analysis.nutrition,
        category: analysis.category,
      });
      
      Alert.alert('¡Análisis completado!', `Detectado: ${analysis.detectedFood}`);
      
    } catch (error) {
      Alert.alert('Error', 'No se pudo analizar la imagen');
    } finally {
      setAnalyzing(false);
    }
  };

  return (
    <Button 
      title={analyzing ? "Analizando..." : "Analizar con IA"} 
      onPress={handleAnalyze}
      disabled={analyzing}
    />
  );
}
```

---

### Ejemplo 3: Estado Persistente

```tsx
import { usePersistentState } from '@/hooks/usePersistentState';

export default function MealsScreen() {
  const [meals, setMeals] = usePersistentState('daily_meals', []);

  const addMeal = (meal) => {
    setMeals([...meals, { ...meal, id: Date.now().toString() }]);
  };

  const removeMeal = (id) => {
    setMeals(meals.filter(m => m.id !== id));
  };

  const updateMeal = (id, updates) => {
    setMeals(meals.map(m => m.id === id ? { ...m, ...updates } : m));
  };

  return (
    <View>
      {meals.map(meal => (
        <MealCard
          key={meal.id}
          meal={meal}
          onEdit={() => handleEdit(meal)}
          onDelete={() => removeMeal(meal.id)}
        />
      ))}
    </View>
  );
}
```

---

## Recursos Adicionales

- [Documentación Completa](./README.md)
- [Arquitectura](./ARCHITECTURE.md)
- [Componentes](./COMPONENTS.md)
- [Servicios](./SERVICES.md)
- [Contribución](./CONTRIBUTING.md)
- [Despliegue](./DEPLOYMENT.md)

---

**Última actualización**: Diciembre 2025
