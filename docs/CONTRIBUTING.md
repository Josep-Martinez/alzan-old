# Guía de Contribución - Alzan

## Índice
1. [Bienvenida](#bienvenida)
2. [Configuración del Entorno](#configuración-del-entorno)
3. [Flujo de Trabajo](#flujo-de-trabajo)
4. [Estándares de Código](#estándares-de-código)
5. [Estructura de Commits](#estructura-de-commits)
6. [Pull Requests](#pull-requests)
7. [Testing](#testing)
8. [Documentación](#documentación)

---

## Bienvenida

¡Gracias por tu interés en contribuir a **Alzan**! 🎉

Este documento te guiará a través del proceso de contribución, desde la configuración inicial hasta el envío de tu primera pull request.

### Valores del Proyecto

- **Calidad sobre cantidad**: Preferimos código bien pensado y testeado
- **Experiencia de usuario**: Cada feature debe mejorar la UX
- **TypeScript estricto**: Sin `any`, sin excepciones
- **Documentación completa**: El código debe ser autoexplicativo

---

## Configuración del Entorno

### Requisitos Previos

- **Node.js**: 18.x o superior (LTS recomendado)
- **pnpm**: 10.24.0 (ver `packageManager` en `package.json`)
- **Git**: Última versión estable
- **Expo CLI**: Se instalará automáticamente
- **iOS Simulator** (Mac) o **Android Studio** (opcional)

### Instalación

**1. Clonar el repositorio**
```bash
git clone https://github.com/tu-usuario/alzan.git
cd alzan
```

**2. Instalar dependencias**
```bash
pnpm install
```

> [!NOTE]
> Este proyecto usa **pnpm** como gestor de paquetes. Si no lo tienes instalado:
> ```bash
> npm install -g pnpm
> ```

**3. Configurar variables de entorno**
```bash
cp .env.example .env
```

Edita `.env` y añade tus claves:
```bash
GEMINI_API_KEY=tu_clave_aquí
```

**4. Iniciar servidor de desarrollo**
```bash
pnpm start
```

Expo te mostrará opciones para abrir en:
- **i**: iOS Simulator
- **a**: Android Emulator
- **w**: Web Browser

---

## Flujo de Trabajo

### 1. Crear una Issue

Antes de empezar a codear, crea una issue describiendo:
- **Problema/Feature**: ¿Qué quieres resolver/añadir?
- **Solución propuesta**: ¿Cómo planeas hacerlo?
- **Alternativas consideradas**: ¿Qué otras opciones evaluaste?

**Plantilla de Issue**:
```markdown
## Descripción
[Describe la feature o bug]

## Propuesta
[Cómo planeas implementarlo]

## Tareas
- [ ] Tarea 1
- [ ] Tarea 2

## Preguntas
- ¿Necesita cambios en la DB?
- ¿Afecta a componentes existentes?
```

---

### 2. Crear una Rama

```bash
# Desde main, crea una rama descriptiva
git checkout -b feature/nombre-descriptivo
# o
git checkout -b fix/bug-que-arregla
```

**Convención de nombres**:
- `feature/` - Nueva funcionalidad
- `fix/` - Corrección de bug
- `refactor/` - Refactorización sin cambios en comportamiento
- `docs/` - Solo cambios en documentación
- `test/` - Añadir o actualizar tests

**Ejemplos**:
```bash
git checkout -b feature/food-favorites
git checkout -b fix/timer-not-stopping
git checkout -b refactor/extract-meal-card-logic
```

---

### 3. Desarrollar

**Ciclo de desarrollo**:
1. Edita código en tu editor favorito
2. Guarda cambios (hot reload automático)
3. Verifica en simulador/emulador
4. Ejecuta linter: `pnpm lint`
5. Fix automático: `pnpm lint --fix`

**Tips**:
- Usa componentes pequeños y enfocados
- Extrae lógica a custom hooks
- Añade comentarios para lógica compleja
- Usa TypeScript sin `any`

---

### 4. Commits

Usa **Conventional Commits**:

```bash
git commit -m "tipo(ámbito): descripción"
```

**Tipos**:
- `feat`: Nueva funcionalidad
- `fix`: Corrección de bug
- `refactor`: Refactorización
- `docs`: Documentación
- `style`: Formato (sin cambios de lógica)
- `test`: Tests
- `chore`: Tareas de mantenimiento

**Ámbitos comunes**:
- `sport`, `nutrition`, `stats`, `ui`, `services`

**Ejemplos**:
```bash
git commit -m "feat(nutrition): add AI food analysis"
git commit -m "fix(sport): timer not stopping after rest period"
git commit -m "refactor(ui): extract Button to separate component"
git commit -m "docs(services): add GeminiFoodAnalyzer documentation"
```

**Descripción**:
- Imperativo: "add" no "added"
- Minúsculas: "add feature" no "Add feature"
- Sin punto final
- Máximo 72 caracteres

---

### 5. Push

```bash
git push origin feature/nombre-descriptivo
```

---

### 6. Pull Request

Crea una PR en GitHub con la siguiente plantilla:

```markdown
## Descripción
[Describe qué hace esta PR]

## Issue Relacionada
Closes #123

## Tipo de Cambio
- [ ] Bug fix
- [ ] Nueva feature
- [ ] Breaking change
- [ ] Documentación

## Checklist
- [ ] He ejecutado `pnpm lint`
- [ ] He probado en iOS/Android
- [ ] He añadido tests (si aplica)
- [ ] He actualizado la documentación
- [ ] Los cambios no rompen features existentes

## Screenshots
[Si hay cambios visuales, añade capturas]

## Testing
[Describe cómo testear los cambios]
```

---

## Estándares de Código

### TypeScript

**✅ Bueno**:
```typescript
interface MealCardProps {
  meal: Meal;
  onEdit: () => void;
  onDelete: () => void;
}

const MealCard: React.FC<MealCardProps> = ({ meal, onEdit, onDelete }) => {
  // ...
};
```

**❌ Malo**:
```typescript
const MealCard = (props: any) => { // ❌ No usar 'any'
  const { meal, onEdit, onDelete } = props;
  // ...
};
```

---

### Nombres Descriptivos

**✅ Bueno**:
```typescript
const handleCompleteWorkout = () => { ... };
const isWorkoutActive = workoutStatus === 'active';
const totalCalories = meals.reduce((sum, m) => sum + m.calories, 0);
```

**❌ Malo**:
```typescript
const handle = () => { ... };
const flag = status === 'active';
const total = meals.reduce((s, m) => s + m.c, 0);
```

---

### Componentes

**Estructura recomendada**:
```typescript
// 1. Imports
import { useState } from 'react';
import { View, Text } from 'react-native';
import { Button } from '@/components/ui/Button';

// 2. Interfaces
interface Props {
  title: string;
  onPress: () => void;
}

// 3. Componente
export const MyComponent: React.FC<Props> = ({ title, onPress }) => {
  // 3a. Estados
  const [isLoading, setLoading] = useState(false);
  
  // 3b. Efectos
  useEffect(() => {
    // ...
  }, []);
  
  // 3c. Funciones
  const handlePress = async () => {
    setLoading(true);
    await onPress();
    setLoading(false);
  };
  
  // 3d. Render
  return (
    <View>
      <Text>{title}</Text>
      <Button onPress={handlePress} loading={isLoading} />
    </View>
  );
};

// 4. Estilos (si usa StyleSheet)
const styles = StyleSheet.create({
  // ...
});
```

---

### Estilos

**Preferencia**:
1. `StyleSheet.create()` para estilos estáticos
2. Inline styles solo para valores dinámicos

**Buenas prácticas**:
```typescript
// ✅ Nombres descriptivos
const styles = StyleSheet.create({
  container: { ... },
  headerText: { ... },
  primaryButton: { ... }
});

// ❌ Nombres genéricos
const styles = StyleSheet.create({
  view1: { ... },
  text1: { ... },
  btn: { ... }
});
```

---

### Comentarios

**Cuándo comentar**:
- Lógica compleja no obvia
- Workarounds temporales
- TODOs con contexto

**✅ Bueno**:
```typescript
// Calculamos calorías usando ecuación de Harris-Benedict
// Fuente: https://example.com/harris-benedict
const bmr = 10 * weight + 6.25 * height - 5 * age + s;

// TODO(josep): Migrar a AsyncStorage cuando tengamos backend
const tempStorage = new Map();
```

**❌ Malo**:
```typescript
// Incrementar i
i++;

// Esta función suma dos números
const add = (a, b) => a + b;
```

---

### Imports

**Orden**:
1. React/React Native
2. Librerías externas
3. Componentes locales
4. Utilidades/servicios
5. Tipos/interfaces
6. Estilos

**Ejemplo**:
```typescript
// React
import { useState, useEffect } from 'react';
import { View, Text } from 'react-native';

// Librerías
import { LinearGradient } from 'expo-linear-gradient';
import { MaterialIcons } from '@expo/vector-icons';

// Componentes
import { Button } from '@/components/ui/Button';
import { Card } from '@/components/ui/Card';

// Servicios
import { GeminiFoodAnalyzer } from '@/components/services/GeminiFoodAnalyzer';

// Tipos
import type { Meal, Food } from '@/types';

// Estilos
import { styles } from './styles';
```

---

## Estructura de Commits

### Commit Atómico

Cada commit debe ser **una unidad lógica** de cambio.

**✅ Bueno** (commits separados):
```bash
git commit -m "feat(ui): add Button component"
git commit -m "feat(nutrition): implement MealCard"
git commit -m "docs(ui): add Button usage examples"
```

**❌ Malo** (todo en uno):
```bash
git commit -m "add button, meal card, and docs"
```

---

### Mensaje Detallado

Para cambios complejos, usa el cuerpo del commit:

```bash
git commit

# Editor se abrirá:
feat(services): add food analysis with Gemini AI

- Implement GeminiFoodAnalyzer service
- Add image to base64 conversion
- Create optimized prompt for nutrition data
- Add validation for API responses

Closes #45
```

**Formato**:
```
<tipo>(<ámbito>): <descripción corta>

[Cuerpo opcional con más detalles]

[Footer opcional: refs, closes, breaking changes]
```

---

## Pull Requests

### Antes de Crear la PR

**Checklist**:
- [ ] `pnpm lint` sin errores
- [ ] Probado en iOS y Android (o al menos web)
- [ ] Commits siguen Conventional Commits
- [ ] Branch actualizada con `main`
- [ ] Sin archivos innecesarios (node_modules, .DS_Store)

**Rebase si es necesario**:
```bash
git fetch origin
git rebase origin/main
```

---

### Descripción Clara

**Incluye**:
- **Qué** hace la PR
- **Por qué** es necesario
- **Cómo** funciona (si no es obvio)
- **Screenshots** (si hay cambios visuales)

**Ejemplo**:

> ## Añadir análisis de comida con IA
> 
> Implementa análisis automático de imágenes de comida usando Google Gemini API.
> 
> ### Cambios
> - Nuevo servicio `GeminiFoodAnalyzer`
> - Componente `AddFoodModalAI` con captura de imagen
> - Parsing de respuesta JSON de Gemini
> - Validación de datos nutricionales
> 
> ### Testing
> 1. Ir a "Nutrición"
> 2. Tocar "Añadir con IA"
> 3. Seleccionar imagen de comida
> 4. Verificar que pre-rellena los macros
> 
> ### Screenshots
> ![Demo](./screenshots/ai-analysis.gif)
> 
> Closes #45

---

### Code Review

**Como autor**:
- Responde a comentarios de manera constructiva
- Haz cambios solicitados en nuevos commits
- Marca conversaciones como resueltas

**Como revisor**:
- Sé respetuoso y constructivo
- Sugiere mejoras, no impongas estilo personal
- Aprueba si está bien, aunque no sea perfecto

---

## Testing

> [!NOTE]
> Actualmente el proyecto **no tiene tests configurados**.  
> Esto es una prioridad pendiente.

### Testing Futuro

**Herramientas planificadas**:
- **Jest**: Unit tests
- **React Native Testing Library**: Component tests
- **Detox**: E2E tests

**Instalación (cuando se implemente)**:
```bash
pnpm add -D jest @testing-library/react-native
```

**Ejemplo de test**:
```typescript
// MealCard.test.tsx
import { render, fireEvent } from '@testing-library/react-native';
import { MealCard } from './MealCard';

describe('MealCard', () => {
  it('should render meal name', () => {
    const meal = { name: 'Desayuno', foods: [], totalMacros: {} };
    const { getByText } = render(<MealCard meal={meal} />);
    
    expect(getByText('Desayuno')).toBeTruthy();
  });
  
  it('should call onEdit when edit button pressed', () => {
    const onEdit = jest.fn();
    const { getByTestId } = render(<MealCard meal={meal} onEdit={onEdit} />);
    
    fireEvent.press(getByTestId('edit-button'));
    
    expect(onEdit).toHaveBeenCalled();
  });
});
```

---

## Documentación

### Códigos que Requieren Docs

**Siempre documenta**:
- Nuevos servicios/integraciones
- Componentes públicos reutilizables
- Funciones con lógica compleja
- APIs/interfaces públicas

**Formato JSDoc**:
```typescript
/**
 * Analiza una imagen de comida y devuelve información nutricional
 * 
 * @param imageUri - URI de la imagen a analizar
 * @returns Información nutricional detectada
 * @throws {Error} Si la imagen no es válida o la API falla
 * 
 * @example
 * ```typescript
 * const result = await analyzeFood('file:///path/to/image.jpg');
 * console.log(result.nutrition.calories); // 450
 * ```
 */
export async function analyzeFood(imageUri: string): Promise<FoodAnalysisResult> {
  // ...
}
```

---

### Actualizar README

Si tu cambio afecta:
- **Setup/instalación**: Actualiza README.md
- **Nueva feature importante**: Añade a "Funcionalidades clave"
- **Nuevo script**: Actualiza "Scripts disponibles"

---

### Documentación en `/docs`

Para cambios arquitectónicos o nuevos módulos:
- **ARCHITECTURE.md**: Si cambias estructura del proyecto
- **COMPONENTS.md**: Si creas componentes reutilizables
- **SERVICES.md**: Si añades servicios/integraciones
- **DEPLOYMENT.md**: Si cambias proceso de deploy

---

## Preguntas Frecuentes

### ¿Puedo usar JavaScript en lugar de TypeScript?
No. Todo el código debe ser TypeScript.

### ¿Qué pasa si rompo algo sin querer?
No pasa nada, para eso están las PRs y el code review. Aprende del error y sigue adelante.

### ¿Cuánto tiempo tarda en revisarse una PR?
Depende de la complejidad, pero normalmente en 1-3 días.

### ¿Puedo trabajar en múltiples features a la vez?
Sí, pero crea ramas separadas para cada una.

### ¿Necesito permiso para empezar a trabajar en una issue?
No si es algo pequeño. Para features grandes, comenta en la issue primero.

---

## Recursos Útiles

### Documentación Externa

- [React Native Docs](https://reactnative.dev/docs/getting-started)
- [Expo Docs](https://docs.expo.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Conventional Commits](https://www.conventionalcommits.org/)

### Herramientas Recomendadas

- **VS Code** con extensiones:
  - ESLint
  - Prettier
  - TypeScript and JavaScript Language Features
  - React Native Tools
- **Expo Go** app (iOS/Android) para testing rápido

---

## Código de Conducta

### Nuestros Estándares

- ✅ Ser respetuoso con todos
- ✅ Aceptar críticas constructivas
- ✅ Enfocarse en lo mejor para el proyecto
- ✅ Mostrar empatía hacia otros contribuidores

- ❌ Comentarios ofensivos o discriminatorios
- ❌ Trolling o ataques personales
- ❌ Publicar información privada de otros

---

## Contacto

- **Issues**: [GitHub Issues](https://github.com/tu-usuario/alzan/issues)
- **Discusiones**: [GitHub Discussions](https://github.com/tu-usuario/alzan/discussions)
- **Email**: alzan-dev@example.com

---

¡Gracias por contribuir a Alzan! 🚀💪🥗
