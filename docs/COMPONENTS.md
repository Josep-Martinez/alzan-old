# Guía de Componentes de Alzan

## Índice
1. [Componentes Deportivos](#componentes-deportivos)
2. [Componentes Nutricionales](#componentes-nutricionales)
3. [Componentes de Estadísticas](#componentes-de-estadísticas)
4. [Componentes UI Base](#componentes-ui-base)
5. [Guía de Uso y Ejemplos](#guía-de-uso-y-ejemplos)

---

## Componentes Deportivos

### 📁 components/sport/gym/

#### ActiveWorkoutSession.tsx
**Propósito**: Gestión de sesiones de entrenamiento activas en tiempo real

**Props**:
```typescript
interface ActiveWorkoutSessionProps {
  exercises: Exercise[];
  onComplete: (summary: WorkoutSummary) => void;
  onCancel: () => void;
}

interface Exercise {
  id: string;
  name: string;
  sets: number;
  reps: number;
  weight?: number;
  restTime?: number; // segundos
}
```

**Estado interno**:
- `currentExerciseIndex`: Ejercicio actual
- `completedSets`: Array de sets completados
- `isResting`: Si está en período de descanso
- `restTimeRemaining`: Tiempo de descanso restante

**Funcionalidades**:
- ✅ Navegación entre ejercicios
- ✅ Registro de sets completados
- ✅ Timer automático de descanso
- ✅ Edición de peso en tiempo real
- ✅ Vibración al completar set
- ✅ Resumen final de sesión

**Ejemplo de uso**:
```tsx
<ActiveWorkoutSession
  exercises={[
    { 
      id: '1', 
      name: 'Sentadilla', 
      sets: 4, 
      reps: 10, 
      weight: 80,
      restTime: 90 
    },
    { 
      id: '2', 
      name: 'Press Banca', 
      sets: 3, 
      reps: 8, 
      weight: 60 
    }
  ]}
  onComplete={(summary) => {
    console.log('Entreno completado:', summary);
    saveTrening(summary);
  }}
  onCancel={() => router.back()}
/>
```

---

#### ExerciseCard.tsx
**Propósito**: Tarjeta individual de ejercicio con controles interactivos

**Props**:
```typescript
interface ExerciseCardProps {
  exercise: Exercise;
  isActive: boolean;
  onSetComplete: (setNumber: number, weight: number) => void;
  onWeightChange: (newWeight: number) => void;
}
```

**Características**:
- Indicador visual de set actual
- Checkboxes para marcar sets completados
- Input inline para ajustar peso
- Animación al completar set
- Gradiente de fondo según estado

**UI Snapshot**:
```
┌─────────────────────────────┐
│ 🏋️ Sentadilla              │
│ ━━━━━━━━━━━━━━━━━━━━━━━━  │
│ Series: 4 | Reps: 10       │
│ Peso: [80] kg              │
│                             │
│ Set 1: ✓                   │
│ Set 2: ✓                   │
│ Set 3: ⏺ ← Actual          │
│ Set 4: ○                   │
└─────────────────────────────┘
```

---

#### ExerciseSelector.tsx
**Propósito**: Modal para seleccionar ejercicios de una base de datos

**Props**:
```typescript
interface ExerciseSelectorProps {
  visible: boolean;
  onClose: () => void;
  onSelect: (exercises: Exercise[]) => void;
  muscleGroup?: 'chest' | 'back' | 'legs' | 'arms' | 'shoulders' | 'core';
}
```

**Funcionalidades**:
- Búsqueda por nombre
- Filtro por grupo muscular
- Selección múltiple
- Previsualización de ejercicio
- Favoritos del usuario

**Base de datos de ejercicios**:
```typescript
const EXERCISES_DB = [
  { 
    id: 'squat', 
    name: 'Sentadilla', 
    muscle: 'legs',
    difficulty: 'intermediate',
    equipment: 'barbell' 
  },
  { 
    id: 'bench-press', 
    name: 'Press Banca', 
    muscle: 'chest',
    difficulty: 'beginner',
    equipment: 'barbell' 
  },
  // ... más ejercicios
];
```

---

#### GymSessionModal.tsx
**Propósito**: Configuración completa de sesión de gimnasio

**Props**:
```typescript
interface GymSessionModalProps {
  visible: boolean;
  onClose: () => void;
  onStart: (config: SessionConfig) => void;
  initialData?: SessionConfig;
}

interface SessionConfig {
  name: string;
  exercises: Exercise[];
  warmup?: boolean;
  cooldown?: boolean;
  notes?: string;
}
```

**Pasos del wizard**:
1. **Nombre de sesión**: (ej. "Día de pierna A")
2. **Seleccionar ejercicios**: Usar ExerciseSelector
3. **Configurar cada ejercicio**: Sets, reps, peso
4. **Opciones adicionales**: Calentamiento, enfriamiento
5. **Confirmación**: Revisión antes de iniciar

**Ejemplo completo**:
```tsx
<GymSessionModal
  visible={showModal}
  onClose={() => setShowModal(false)}
  onStart={(config) => {
    // Guardar configuración
    saveSessionConfig(config);
    // Iniciar sesión activa
    setActiveSession(config);
  }}
  initialData={savedTemplate} // Pre-cargar plantilla
/>
```

---

#### RestTimerModal.tsx
**Propósito**: Temporizador de descanso entre series

**Props**:
```typescript
interface RestTimerModalProps {
  visible: boolean;
  duration: number; // segundos
  onComplete: () => void;
  onSkip: () => void;
}
```

**Características**:
- Cuenta regresiva visual (circular)
- Botones de ajuste rápido (+15s, -15s)
- Vibración al terminar
- Sonido opcional (configurable)
- Posibilidad de saltar descanso

**UI Visual**:
```
┌─────────────────────┐
│   Descanso          │
│                     │
│       ⏰            │
│      ╱   ╲         │
│     │ 45s │        │
│      ╲   ╱         │
│                     │
│  -15s  ⏭  +15s     │
│                     │
│  [Saltar Descanso]  │
└─────────────────────┘
```

---

#### WorkoutIntensityModal.tsx
**Propósito**: Registro de intensidad percibida post-entrenamiento (RPE)

**Props**:
```typescript
interface WorkoutIntensityModalProps {
  visible: boolean;
  onSubmit: (intensity: IntensityData) => void;
  onClose: () => void;
}

interface IntensityData {
  rpe: number; // 1-10 (Rate of Perceived Exertion)
  difficulty: 'easy' | 'moderate' | 'hard' | 'very_hard';
  notes?: string;
  musclesSore?: string[]; // ["legs", "glutes"]
}
```

**Escala RPE**:
| Valor | Descripción | Color |
|-------|-------------|-------|
| 1-3 | Muy fácil | 🟢 Verde |
| 4-6 | Moderado | 🟡 Amarillo |
| 7-8 | Difícil | 🟠 Naranja |
| 9-10 | Máximo esfuerzo | 🔴 Rojo |

**Ejemplo de uso**:
```tsx
<WorkoutIntensityModal
  visible={showIntensity}
  onSubmit={(data) => {
    // Guardar feedback de sesión
    updateWorkoutLog({
      ...workoutSummary,
      intensity: data
    });
    setShowIntensity(false);
  }}
  onClose={() => setShowIntensity(false)}
/>
```

---

#### SupersetBuilder.tsx
**Propósito**: Constructor de superseries (ejercicios sin descanso intermedio)

**Props**:
```typescript
interface SupersetBuilderProps {
  onSave: (superset: Superset) => void;
  onCancel: () => void;
}

interface Superset {
  id: string;
  name: string;
  exercises: Exercise[];
  restBetweenRounds: number;
  rounds: number;
}
```

**Características**:
- Agregar/remover ejercicios
- Reordenar con drag & drop
- Configurar descanso entre rondas
- Previsualización del circuito

**Ejemplo**:
```typescript
const superset = {
  id: 'ss1',
  name: 'Brazos Superset',
  exercises: [
    { name: 'Curl Bíceps', sets: 3, reps: 12 },
    { name: 'Extensión Tríceps', sets: 3, reps: 12 }
  ],
  restBetweenRounds: 60, // 1 minuto entre rondas
  rounds: 3
};
```

---

### 📁 components/sport/common/

#### SportCard.tsx
**Propósito**: Tarjeta de deporte en planificador semanal

**Props**:
```typescript
interface SportCardProps {
  sport: Sport;
  date: Date;
  onPress: () => void;
  isCompleted: boolean;
}

interface Sport {
  id: string;
  name: string;
  type: 'gym' | 'running' | 'cycling' | 'swimming' | 'other';
  icon: string;
  color: string;
  duration?: number; // minutos
  planificado: boolean;
}
```

**Estados visuales**:
- **Planificado**: Borde azul, icono normal
- **En progreso**: Animación de pulso, borde verde
- **Completado**: ✓ verde, opacidad reducida
- **Perdido**: Borde rojo, icono desactivado

---

#### WeeklyCalendar.tsx
**Propósito**: Vista semanal de planificación deportiva

**Props**:
```typescript
interface WeeklyCalendarProps {
  startDate: Date;
  sessions: SportSession[];
  onDayPress: (date: Date) => void;
  onSessionPress: (session: SportSession) => void;
}
```

**Características**:
- Navegación semana anterior/siguiente
- Indicadores de sesiones planificadas
- Destaque del día actual
- Vista resumida de sesiones por día

**UI Layout**:
```
┌─────────────────────────────────┐
│  ← Semana 48 del 2025 →        │
├───┬───┬───┬───┬───┬───┬───┤
│ L │ M │ X │ J │ V │ S │ D │
├───┼───┼───┼───┼───┼───┼───┤
│ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │
│🏋️ │   │🏃│🏋️ │   │🚴 │   │
└───┴───┴───┴───┴───┴───┴───┘
```

---

#### SessionSummaryModal.tsx
**Propósito**: Resumen detallado de sesión completada

**Props**:
```typescript
interface SessionSummaryModalProps {
  visible: boolean;
  session: CompletedSession;
  onClose: () => void;
  onShare?: () => void;
}

interface CompletedSession {
  sport: string;
  date: Date;
  duration: number;
  exercises?: Exercise[];
  distance?: number;
  calories?: number;
  averageHeartRate?: number;
  notes?: string;
}
```

**Información mostrada**:
- Duración total
- Ejercicios realizados (si es gym)
- Distancia (si es running/cycling)
- Calorías quemadas estimadas
- Frecuencia cardíaca promedio
- Notas del usuario

---

### 📁 components/sport/other/

#### RunningSessionModal.tsx
**Propósito**: Configurar y rastrear sesión de running

**Datos capturados**:
```typescript
interface RunningSession {
  distance: number; // km
  duration: number; // minutos
  pace: string; // min/km (calculado automáticamente)
  elevation?: number; // metros
  route?: string; // nombre de ruta
  type: 'easy' | 'tempo' | 'intervals' | 'long_run';
}
```

**Cálculos automáticos**:
```typescript
// Ritmo = Duración / Distancia
const pace = duration / distance; // ej: 5:30 min/km

// Distancia = Velocidad × Tiempo
const distance = (speed * duration) / 60;
```

**Tipos de entrenamiento**:
- **Easy Run**: Ritmo cómodo, 60-70% FCmax
- **Tempo Run**: Ritmo fuerte sostenido, 80-85% FCmax
- **Intervals**: Series rápidas con descanso
- **Long Run**: Carrera larga, >90 minutos

---

#### CyclingSessionModal.tsx
**Propósito**: Configurar sesión de ciclismo

**Datos capturados**:
```typescript
interface CyclingSession {
  distance: number; // km
  duration: number; // minutos
  avgSpeed: number; // km/h
  maxSpeed: number; // km/h
  elevation: number; // metros ascendidos
  power?: number; // watts (si tiene potenciómetro)
  cadence?: number; // rpm
  terrain: 'flat' | 'hills' | 'mountain';
}
```

---

#### SwimmingSessionModal.tsx
**Propósito**: Configurar sesión de natación

**Datos capturados**:
```typescript
interface SwimmingSession {
  distance: number; // metros
  duration: number; // minutos
  strokes: {
    freestyle: number;
    backstroke: number;
    breaststroke: number;
    butterfly: number;
  };
  laps: number; // largos (25m o 50m)
  poolLength: 25 | 50;
}
```

---

## Componentes Nutricionales

### 📁 components/nutri/

#### MealCard.tsx
**Propósito**: Tarjeta de comida con resumen de macros

**Props**:
```typescript
interface MealCardProps {
  meal: Meal;
  onEdit?: () => void;
  onDelete?: () => void;
  expanded?: boolean;
}

interface Meal {
  id: string;
  name: string; // "Desayuno", "Almuerzo", etc.
  time: string; // "08:00"
  foods: Food[];
  totalMacros: {
    calories: number;
    protein: number;
    carbs: number;
    fats: number;
  };
}

interface Food {
  id: string;
  name: string;
  quantity: number;
  unit: 'g' | 'ml' | 'unit';
  macros: Macros;
  category: 'real' | 'good' | 'ultra';
}
```

**Características**:
- Badge de categoría (🟢 Real, 🟡 Procesada, 🔴 Ultra)
- Resumen visual de macros (barras de progreso)
- Lista expandible de alimentos
- Acciones rápidas (editar, eliminar)

**UI Snapshot**:
```
┌─────────────────────────────┐
│ 🍳 Desayuno - 08:00        │
│ ━━━━━━━━━━━━━━━━━━━━━━━━  │
│ 450 kcal | 30g P | 40g C   │
│          | 15g F           │
│                             │
│ • Avena (50g) 🟢           │
│ • Plátano (1 ud) 🟢        │
│ • Proteína (30g) 🟡        │
│                             │
│ P ██████░░  75%            │
│ C ████████  90%            │
│ F ████░░░░  60%            │
└─────────────────────────────┘
```

---

#### AddFoodModal.tsx
**Propósito**: Modal para añadir comida manualmente

**Props**:
```typescript
interface AddFoodModalProps {
  visible: boolean;
  mealId: string; // A qué comida añadir
  onClose: () => void;
  onSave: (food: Food) => void;
  initialData?: Partial<Food>; // Para edición
}
```

**Campos del formulario**:
1. **Nombre del alimento** (texto)
2. **Cantidad** (número)
3. **Unidad** (g, ml, unidades, porciones)
4. **Macros**:
   - Calorías (auto-calculadas o manual)
   - Proteínas (g)
   - Carbohidratos (g)
   - Grasas (g)
   - Fibra (g) - opcional
   - Azúcares (g) - opcional
5. **Categoría** (comida real, procesada, ultraprocesada)

**Validaciones**:
```typescript
const validateMacros = (macros: Macros) => {
  // Regla: 1g proteína = 4kcal, 1g carbs = 4kcal, 1g fat = 9kcal
  const calculatedCals = 
    (macros.protein * 4) + 
    (macros.carbs * 4) + 
    (macros.fats * 9);
  
  const difference = Math.abs(calculatedCals - macros.calories);
  
  if (difference > 50) {
    console.warn('Calorías no coinciden con macros');
  }
};
```

**Autocompletado**:
- Base de datos local de alimentos comunes
- Últimos alimentos añadidos
- Favoritos del usuario

---

#### AddFoodModalAI.tsx
**Propósito**: Modal con análisis de IA para detectar comida

**Flujo de uso**:
1. Usuario toca botón "Analizar con IA"
2. Se abre cámara o selector de galería
3. Usuario toma/selecciona foto
4. Envío a `GeminiFoodAnalyzer`
5. Muestra loading (spinner + texto motivacional)
6. Respuesta de IA pre-rellena formulario
7. Usuario confirma o ajusta valores
8. Guardar comida

**Props**:
```typescript
interface AddFoodModalAIProps {
  visible: boolean;
  mealId: string;
  onClose: () => void;
  onSave: (food: Food) => void;
}
```

**Estados del modal**:
- `idle`: Esperando acción del usuario
- `capturing`: Tomando/seleccionando foto
- `analyzing`: Enviando a IA
- `reviewing`: Mostrando resultados para confirmar
- `error`: Mostrando error

**Ejemplo de integración**:
```tsx
const handleAnalyze = async (imageUri: string) => {
  setStatus('analyzing');
  
  try {
    const result = await GeminiFoodAnalyzer.analyzeFood(imageUri);
    
    // Pre-rellenar formulario
    setFoodData({
      name: result.detectedFood,
      quantity: parseQuantity(result.estimatedQuantity),
      macros: result.nutrition,
      category: result.category,
      confidence: result.confidence
    });
    
    setStatus('reviewing');
  } catch (error) {
    console.error('Error al analizar:', error);
    setStatus('error');
    Alert.alert('Error', 'No se pudo analizar la imagen');
  }
};
```

---

#### FoodScanner.tsx
**Propósito**: Scanner de código de barras para productos envasados

**Props**:
```typescript
interface FoodScannerProps {
  visible: boolean;
  onScan: (barcode: string) => void;
  onClose: () => void;
}
```

**Flujo de escaneo**:
1. Solicitar permiso de cámara
2. Activar scanner (expo-camera)
3. Detectar código de barras (UPC, EAN13, QR)
4. Vibración + sonido al detectar
5. Buscar en base de datos de productos
6. Autocompletar información nutricional

**Integración con OpenFoodFacts**:
```typescript
const searchProduct = async (barcode: string) => {
  const response = await fetch(
    `https://world.openfoodfacts.org/api/v0/product/${barcode}.json`
  );
  const data = await response.json();
  
  if (data.status === 1) {
    return {
      name: data.product.product_name,
      macros: {
        calories: data.product.nutriments['energy-kcal_100g'],
        protein: data.product.nutriments.proteins_100g,
        carbs: data.product.nutriments.carbohydrates_100g,
        fats: data.product.nutriments.fat_100g
      },
      category: classifyProduct(data.product.nova_group)
    };
  }
  
  return null;
};
```

---

#### AddExtraMealModal.tsx
**Propósito**: Añadir comidas adicionales fuera del horario estándar

**Casos de uso**:
- Snacks entre comidas
- Post-entreno
- Pre-entreno
- Suplementos

**Props**:
```typescript
interface AddExtraMealModalProps {
  visible: boolean;
  onClose: () => void;
  onSave: (meal: Meal) => void;
}
```

**Diferencias con comidas normales**:
- Horario flexible (cualquier hora)
- Categorización especial (snack, pre/post workout)
- No cuenta para ratio comidas/día en stats

---

## Componentes de Estadísticas

### 📁 components/stats/

#### WeeklyProgressChart.tsx
**Propósito**: Gráfico de progreso semanal (calorías, macros, entrenamientos)

**Props**:
```typescript
interface WeeklyProgressChartProps {
  data: WeeklyData[];
  metric: 'calories' | 'protein' | 'carbs' | 'fats' | 'workouts';
  targetValue?: number;
}

interface WeeklyData {
  date: Date;
  value: number;
  target: number;
}
```

**Tipos de gráficos**:
- Barras verticales para valores diarios
- Línea de tendencia
- Línea de objetivo (horizontal, punteada)

---

#### PersonalRecordsCard.tsx
**Propósito**: Mostrar récords personales (PRs)

**Datos mostrados**:
```typescript
interface PersonalRecord {
  exercise: string;
  value: number;
  unit: 'kg' | 'km' | 'min';
  date: Date;
  improvement?: number; // % mejora respecto anterior PR
}
```

**Ejemplos**:
- 🏋️ Sentadilla: 120kg (↑ 5kg desde 19/11)
- 🏃 5K más rápido: 22:30 (↓ 45s)
- 🚴 Mayor distancia: 85km

---

#### ConsistencyTracker.tsx
**Propósito**: Seguimiento de consistencia (streaks)

**Métricas**:
```typescript
interface ConsistencyMetrics {
  currentStreak: number; // Días consecutivos
  longestStreak: number;
  totalDaysActive: number;
  weeklyAverage: number; // Días activos por semana
  heatmap: DayActivity[]; // Para visualización tipo GitHub
}

interface DayActivity {
  date: Date;
  workouts: number;
  mealsLogged: number;
  active: boolean;
}
```

**Visualización heatmap**:
```
Actividad últimos 3 meses:
┌─────────────────────────────┐
│ L M X J V S D               │
│ ▓ ▓ ░ ▓ ▓ ░ ░  Sem 46      │
│ ▓ ▓ ▓ ▓ ▓ ▓ ░  Sem 47      │
│ ▓ ▓ ▓ ▓ ▓ ░ ░  Sem 48 ← Actual
└─────────────────────────────┘
▓ = Activo | ░ = Inactivo
```

---

#### NutritionSummary.tsx
**Propósito**: Resumen nutricional del día/semana

**Datos mostrados**:
```typescript
interface NutritionSummary {
  period: 'today' | 'week' | 'month';
  totalMeals: number;
  averageCalories: number;
  macroDistribution: {
    protein: number; // %
    carbs: number;
    fats: number;
  };
  realFoodPercentage: number; // % de comida real vs procesada
  waterIntake: number; // litros
}
```

---

#### WorkoutSummary.tsx
**Propósito**: Resumen de entrenamientos

**Datos mostrados**:
```typescript
interface WorkoutSummary {
  period: 'week' | 'month' | 'year';
  totalWorkouts: number;
  totalDuration: number; // minutos
  byType: {
    gym: number;
    running: number;
    cycling: number;
    swimming: number;
    other: number;
  };
  averageIntensity: number; // RPE promedio
  totalVolume?: number; // kg levantados (gym)
  totalDistance?: number; // km (running/cycling)
}
```

---

## Componentes UI Base

### 📁 components/ui/

#### Button.tsx
**Propósito**: Botón reutilizable con variantes

**Props**:
```typescript
interface ButtonProps {
  title: string;
  onPress: () => void;
  variant: 'primary' | 'secondary' | 'outline' | 'ghost';
  size: 'small' | 'medium' | 'large';
  icon?: string;
  loading?: boolean;
  disabled?: boolean;
  fullWidth?: boolean;
}
```

**Variantes**:
- `primary`: Gradiente azul/morado, texto blanco
- `secondary`: Gris, texto blanco
- `outline`: Borde coloreado, fondo transparente
- `ghost`: Sin borde ni fondo, solo texto

---

#### Card.tsx
**Propósito**: Contenedor de contenido con gradiente

**Props**:
```typescript
interface CardProps {
  children: React.ReactNode;
  gradient?: [string, string]; // Colores del gradiente
  padding?: number;
  borderRadius?: number;
  onPress?: () => void;
  shadow?: boolean;
}
```

---

#### Input.tsx
**Propósito**: Input de texto estilizado

**Props**:
```typescript
interface InputProps {
  value: string;
  onChangeText: (text: string) => void;
  placeholder?: string;
  keyboardType?: 'default' | 'numeric' | 'email-address';
  secureTextEntry?: boolean;
  icon?: string;
  error?: string;
  maxLength?: number;
}
```

---

#### Modal.tsx
**Propósito**: Modal base reutilizable

**Props**:
```typescript
interface ModalProps {
  visible: boolean;
  onClose: () => void;
  title?: string;
  children: React.ReactNode;
  size?: 'small' | 'medium' | 'large' | 'fullscreen';
  showCloseButton?: boolean;
}
```

---

## Guía de Uso y Ejemplos

### Ejemplo completo: Flujo de gimnasio

```tsx
import { useState } from 'react';
import GymSessionModal from '@/components/sport/gym/GymSessionModal';
import ActiveWorkoutSession from '@/components/sport/gym/ActiveWorkoutSession';
import WorkoutIntensityModal from '@/components/sport/gym/WorkoutIntensityModal';

export default function WorkoutScreen() {
  const [showConfig, setShowConfig] = useState(false);
  const [activeSession, setActiveSession] = useState(null);
  const [showIntensity, setShowIntensity] = useState(false);
  const [workoutSummary, setWorkoutSummary] = useState(null);

  // 1. Configurar sesión
  const handleConfigSession = (config) => {
    setShowConfig(false);
    setActiveSession(config);
  };

  // 2. Completar entreno
  const handleCompleteWorkout = (summary) => {
    setActiveSession(null);
    setWorkoutSummary(summary);
    setShowIntensity(true);
  };

  // 3. Registrar intensidad
  const handleIntensitySubmit = (intensityData) => {
    const completeData = {
      ...workoutSummary,
      intensity: intensityData
    };
    
    // Guardar en base de datos
    saveWorkoutToDB(completeData);
    
    setShowIntensity(false);
    
    // Mostrar resumen final
    Alert.alert('¡Entreno completado!', 'Buen trabajo 💪');
  };

  return (
    <View>
      {/* Botón para iniciar */}
      <Button 
        title="Nuevo Entreno" 
        onPress={() => setShowConfig(true)} 
      />

      {/* Modal de configuración */}
      <GymSessionModal
        visible={showConfig}
        onClose={() => setShowConfig(false)}
        onStart={handleConfigSession}
      />

      {/* Sesión activa */}
      {activeSession && (
        <ActiveWorkoutSession
          exercises={activeSession.exercises}
          onComplete={handleCompleteWorkout}
          onCancel={() => setActiveSession(null)}
        />
      )}

      {/* Modal de intensidad */}
      <WorkoutIntensityModal
        visible={showIntensity}
        onSubmit={handleIntensitySubmit}
        onClose={() => setShowIntensity(false)}
      />
    </View>
  );
}
```

---

### Ejemplo completo: Flujo de nutrición con IA

```tsx
import { useState } from 'react';
import MealCard from '@/components/nutri/MealCard';
import AddFoodModalAI from '@/components/nutri/AddFoodModalAI';

export default function NutritionScreen() {
  const [meals, setMeals] = useState([
    { id: '1', name: 'Desayuno', time: '08:00', foods: [], totalMacros: {} },
    { id: '2', name: 'Almuerzo', time: '14:00', foods: [], totalMacros: {} },
    { id: '3', name: 'Cena', time: '21:00', foods: [], totalMacros: {} },
  ]);
  const [selectedMeal, setSelectedMeal] = useState(null);
  const [showAIModal, setShowAIModal] = useState(false);

  const handleAddFood = (mealId) => {
    setSelectedMeal(mealId);
    setShowAIModal(true);
  };

  const handleSaveFood = (food) => {
    setMeals(prev => prev.map(meal => {
      if (meal.id === selectedMeal) {
        const newFoods = [...meal.foods, food];
        const totalMacros = calculateTotalMacros(newFoods);
        return { ...meal, foods: newFoods, totalMacros };
      }
      return meal;
    }));
    
    setShowAIModal(false);
  };

  const calculateTotalMacros = (foods) => {
    return foods.reduce((acc, food) => ({
      calories: acc.calories + food.macros.calories,
      protein: acc.protein + food.macros.protein,
      carbs: acc.carbs + food.macros.carbs,
      fats: acc.fats + food.macros.fats
    }), { calories: 0, protein: 0, carbs: 0, fats: 0 });
  };

  return (
    <ScrollView>
      {meals.map(meal => (
        <MealCard
          key={meal.id}
          meal={meal}
          onEdit={() => handleAddFood(meal.id)}
        />
      ))}

      <AddFoodModalAI
        visible={showAIModal}
        mealId={selectedMeal}
        onClose={() => setShowAIModal(false)}
        onSave={handleSaveFood}
      />
    </ScrollView>
  );
}
```

---

## Mejores Prácticas

### 1. Composición sobre herencia
✅ **Bueno**:
```tsx
<Card>
  <CardHeader title="Desayuno" />
  <CardContent>
    <FoodList foods={foods} />
  </CardContent>
</Card>
```

❌ **Malo**:
```tsx
class BreakfastCard extends Card {
  // ...
}
```

### 2. Props drilling limitado
Si pasas props a través de más de 3 niveles, considera Context o estado global.

### 3. Memoización para performance
```tsx
const MealCard = React.memo(({ meal }) => {
  // ... renderizado
}, (prevProps, nextProps) => {
  return prevProps.meal.id === nextProps.meal.id;
});
```

### 4. Custom hooks para lógica compartida
```tsx
// useMealTracking.ts
export const useMealTracking = () => {
  const [meals, setMeals] = useState([]);
  
  const addMeal = (meal) => { ... };
  const removeMeal = (id) => { ... };
  const calculateDailyTotals = () => { ... };
  
  return { meals, addMeal, removeMeal, calculateDailyTotals };
};

// En componente
const { meals, addMeal } = useMealTracking();
```

---

## Próximos Componentes

Componentes planificados para futuras versiones:

- [ ] `SocialShareCard`: Compartir progreso en redes
- [ ] `MealPlanner`: Planificador semanal de comidas
- [ ] `RecipeBook`: Recetario con valores nutricionales
- [ ] `LeaderboardCard`: Ranking entre amigos
- [ ] `AchievementBadge`: Sistema de logros
- [ ] `ProgressPhotos`: Galería de fotos de progreso

---

Para más información, consulta:
- [Arquitectura](./ARCHITECTURE.md)
- [Servicios e Integraciones](./SERVICES.md)
- [Guía de Contribución](./CONTRIBUTING.md)
