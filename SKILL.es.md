name: algo-alchemist
version: "1.2.0"
author: Kilocode Team
description: Transforma algoritmos crudos en soluciones elegantes y óptimas mediante análisis de rendimiento sistemático, reducción de complejidad y refinamiento de código.
main: bin/algo-alchemist
entrypoint: algo-alchemist
tags:
  - optimization
  - algorithms
  - performance
  - refactoring
  - complexity
language: python
dependencies:
  - python>=3.8
  - mypy
  - pytest
  - timeit
  - astunparse
  - networkx
  - numpy
optional_dependencies:
  - py-spy
  - memory_profiler
  - pyperf
  - tqdm
environment:
  ALGO_CACHE_DIR: "~/.cache/algo-alchemist"
  ALGO_TIMEOUT: "30"
  ALGO_VERBOSE: "0"
  ALGO_MAX_ITERATIONS: "10"
---
# Algo Alchemist

Transforma algoritmos crudos en soluciones elegantes y óptimas mediante análisis de rendimiento sistemático, reducción de complejidad y refinamiento de código.

## Propósito

Casos de uso reales:
- Mejorar la complejidad tiempo/espacio de código existente (O(n²) → O(n log n) o mejor)
- Transformar implementaciones ingenuas en soluciones eficientes y listas para producción
- Optimizar cuellos de botella críticos de rendimiento en código sensible a latencia
- Refactorizar código espagueti en algoritmos limpios y mantenibles
- Convertir entre paradigmas algorítmicos (recursivo ↔ iterativo, DP ↔ codicioso)
- Aplicar técnicas de memoización, programación dinámica y divide-y-vencerás
- Optimizar estructuras de datos para patrones de acceso específicos (búsqueda intensiva vs escritura intensiva)
- Reducir la huella de memoria mediante operaciones in-situ o algoritmos de streaming

## Alcance

Lenguajes compatibles: Python 3.8+, JavaScript (Node.js 14+), TypeScript 4.0+

### Comandos

`algo-alchemist analyze <file> [--function <name>] [--profile]`
Analiza la complejidad algorítmica, identifica cuellos de botella, opcionalmente perfila el tiempo de ejecución. Genera análisis de complejidad, informe de hotspots y sugerencias de mejora.

`algo-alchemist optimize <file> --target-complexity <O(n)|O(n log n)|O(1)> [--preserve-api] [--max-iterations <int>]`
Aplica transformaciones automatizadas para lograr la complejidad objetivo. Prueba y valida iterativamente las mejoras.

`algo-alchemist transform <file> --paradigm <iterative|recursive|dynamic|greedy> [--function <name>]`
Convierte el algoritmo al paradigma especificado preservando la semántica.

`algo-alchemist memoize <file> --function <name> [--strategy <lru|custom|dp>]`
Añade memoización/caché a cálculos recursivos o repetitivos.

`algo-alchemist datastruct <file> --optimize <lookup|insertion|iteration> --constraints <memory|speed>`
Recomienda y aplica optimizaciones de estructuras de datos (list → dict, array → set, etc.)

`algo-alchemist profile <file> --input <test-data> [--runs <int>] [--memory]`
Ejecuta micropruebas de rendimiento con los datos de entrada proporcionados. Genera perfiles de rendimiento.

`algo-alchemist suggest <file> [--function <name>]`
Devuelve sugerencias de optimización accionables sin aplicar cambios.

`algo-alchemist apply <file> --suggestion-id <id> [--dry-run]`
Aplica una sugerencia de optimización específica de la salida de suggest.

`algo-alchemist verify <file> [--tests <path>] [--complexity-check]`
Valida la corrección contra pruebas y verifica mejoras de complejidad.

`algo-alchemist undo <file> [--steps <int>]`
Revierte las últimas N transformaciones (por defecto: 1). Mantiene el historial de transformaciones.

`algo-alchemist history <file>`
Muestra el registro completo de transformaciones con diffs y cambios de complejidad.

`algo-alchemist benchmark <file> --baseline <commit|branch> --input <test-data>`
Compara el rendimiento contra una línea base. Requiere py-spy o pyperf para perfilado detallado.

### Ejemplos de Prompts Reales

```bash
# Analyze nested loops in find_duplicates.py
algo-alchemist analyze find_duplicates.py --function find_pairs

# Optimize to O(n) using hash map
algo-alchemist optimize find_duplicates.py --target-complexity O(n) --preserve-api

# Convert recursive DFS to iterative stack-based
algo-alchemist transform maze_solver.py --paradigm iterative --function dfs

# Add memoization to expensive recursive calculation
algo-alchemist memoize fibonacci.py --function fib --strategy lru

# Optimize list-based membership checks to set-based
algo-alchemist datastruct filter_items.py --optimize lookup --constraints speed

# Profile with real dataset
algo-alchemist analyze sort_compare.py --profile --input tests/data/large_array.json

# Get suggestions without applying
algo-alchemist suggest network_routing.py --function shortest_path

# Apply suggestion #3 only
algo-alchemist apply network_routing.py --suggestion-id 3 --dry-run

# Verify correctness after optimization
algo-alchemist verify optimized_sort.py --tests tests/test_sorts.py --complexity-check
```

## Proceso de Trabajo Detallado

### 1. Análisis Inicial
- Analizar archivo fuente usando AST
- Identificar función(es) objetivo o análisis de archivo completo
- Calcular la complejidad temporal y espacial Big O actual
- Detectar anti-patrones: bucles anidados, computaciones repetidas, búsquedas lineales en bucles, copias innecesarias
- Generar métricas de línea base si el perfilado está habilitado

### 2. Perfilado (Opcional)
- Ejecutar con los datos de entrada proporcionados o generar casos de prueba sintéticos
- Medir tiempo real, uso de CPU y asignaciones de memoria
- Identificar hotspots (funciones que consumen >50% del tiempo de ejecución)
- Perfilar asignaciones de memoria con memory_profiler si está disponible

### 3. Planificación de Optimización
- Enumerar transformaciones aplicables basadas en patrones detectados
- Estimar la reducción de complejidad para cada transformación
- Priorizar por impacto y riesgo (preservación de corrección)
- Generar plan de transformación (hasta `ALGO_MAX_ITERATIONS` pasos)

### 4. Aplicación de Transformación
- Aplicar transformaciones secuencialmente con validación entre pasos
- Cada transformación se registra en el archivo de historial `.algo-history.json`
- El código se reformatea para coincidir con el estilo existente (black para Python, prettier para JS/TS)
- Verificación de tipos con mypy/tsc para asegurar que no hay regresiones de tipo

### 5. Verificación
- Ejecutar suite de pruebas existente (pytest/jest) para asegurar equivalencia semántica
- Reanalizar complejidad para confirmar la mejora
- Ejecutar comparación de benchmarks si hay línea base disponible
- Verificar regresiones de legibilidad (conteo de líneas, complejidad ciclomática)

### 6. Iteración
- Si no se alcanza la complejidad objetivo, repetir desde el paso 3 con las transformaciones restantes
- Si las pruebas fallan, deshacer automáticamente la última transformación y probar alternativa
- Abortar después de `ALGO_MAX_ITERATIONS` si no hay transformaciones exitosas

### 7. Reporte Final
- Resumen de transformaciones aplicadas
- Comparación de complejidad antes/después
- Métricas de mejora de rendimiento (si se perfiló)
- Diff de cambios de código
- Lista de cualquier elemento de revisión manual

## Reglas de Oro

1. **Primero la Corrección**: Nunca sacrifiques la corrección funcional por rendimiento. Todas las pruebas deben pasar.
2. **Estabilidad de API**: Las firmas de funciones públicas y los tipos de retorno deben permanecer inalterados a menos que `--preserve-api` sea falso.
3. **Legibilidad Preservada**: El código optimizado no debe aumentar la complejidad ciclomática. Prefiere la claridad sobre el ingenio.
4. **Basado en Perfilado**: No apliques transformaciones sin datos de perfilado cuando `--profile` esté disponible. Evita la optimización prematura.
5. **Transformaciones Atómicas**: Cada cambio debe estar aislado y ser reversible. Nunca mezcles múltiples optimizaciones en un solo commit.
6. **Dependencias Explícitas**: Rastrea las dependencias añadidas (por ejemplo, functools.lru_cache es parte de la biblioteca estándar, pero las librerías externas deben ser anotadas).
7. **Conciencia de Memoria**: Considera explícitamente los trade-offs espacio-tiempo. Documenta si la optimización aumenta el uso de memoria.
8. **Casos Extremo Validados**: Asegúrate de que el algoritmo transformado maneje entradas vacías, entradas grandes y condiciones de frontera de manera idéntica.

## Ejemplos

### Ejemplo 1: O(n²) a O(n) con Hash Map

**Archivo de entrada `find_duplicates.py` (antes):**
```python
def find_duplicates(arr):
  duplicates = []
  for i in range(len(arr)):
    for j in range(i + 1, len(arr)):
      if arr[i] == arr[j] and arr[i] not in duplicates:
        duplicates.append(arr[i])
  return duplicates
```

**Comando:**
`algo-alchemist optimize find_duplicates.py --target-complexity O(n)`

**Salida (después):**
```python
def find_duplicates(arr):
  seen = set()
  duplicates = set()
  for item in arr:
    if item in seen:
      duplicates.add(item)
    else:
      seen.add(item)
  return list(duplicates)
```

**Verificación:**
```
$ algo-alchemist verify find_duplicates.py
✓ All tests passed (5/5)
✓ Complexity: O(n²) → O(n)
✓ Memory: +O(n) for hash set (acceptable tradeoff)
```

### Ejemplo 2: Fibonacci Recursivo → Con Memoización

**Entrada `fibonacci.py`:**
```python
def fib(n):
  if n <= 1:
    return n
  return fib(n-1) + fib(n-2)
```

**Comando:**
`algo-alchemist memoize fibonacci.py --function fib --strategy lru`

**Salida:**
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
  if n <= 1:
    return n
  return fib(n-1) + fib(n-2)
```

**Comparación de perfil:**
```
Before: fib(35) = 9227465 (15.2s)
After:  fib(35) = 9227465 (0.004s)
Speedup: 3800×
```

### Ejemplo 3: DFS Recursivo → Iterativo

**Entrada `maze_solver.py` (DFS recursivo):**
```python
def solve(maze, start, end):
  visited = set()
  def dfs(pos):
    if pos == end:
      return True
    visited.add(pos)
    for neighbor in maze.neighbors(pos):
      if neighbor not in visited and dfs(neighbor):
        return True
    return False
  return dfs(start)
```

**Comando:**
`algo-alchemist transform maze_solver.py --paradigm iterative --function solve`

**Salida (iterativa con pila explícita):**
```python
def solve(maze, start, end):
  visited = set()
  stack = [start]
  while stack:
    pos = stack.pop()
    if pos == end:
      return True
    if pos not in visited:
      visited.add(pos)
      for neighbor in maze.neighbors(pos):
        if neighbor not in visited:
          stack.append(neighbor)
  return False
```

## Comandos de Reversión

- `algo-alchemist undo <file> [--steps N]` — Revertir las últimas N transformaciones. Ejemplo: `algo-alchemist undo optimized.py --steps 2`
- `algo-alchemist restore <file> --to <commit-hash>` — Restaurar a un commit Git específico o snapshot del historial. Ejemplo: `algo-alchemist restore algo.py --to abc1234`
- `algo-alchemist history <file>` — Mostrar registro de transformaciones con marcas de tiempo, diffs y cambios de complejidad.
- `algo-alchemist diff <file> --version <N>` — Comparar la versión actual con la versión N en el historial. Ejemplo: `algo-alchemist diff optimized.py --version 3`

**Importante**: El historial se almacena en `.algo-history.json` en el mismo directorio. Respaldar este archivo si se necesita rollback persistente entre sesiones.

## Pasos de Verificación

Después de cualquier ejecución de optimización:

1. **Corrección**: `algo-alchemist verify <file> --tests <path>` — Ejecutar suite de pruebas completa (requerido).
2. **Complejidad**: `algo-alchemist verify <file> --complexity-check` — Asegurar que Big O mejoró como se esperaba.
3. **Benchmark**: `algo-alchemist benchmark <file> --baseline <original-commit> --input <dataset>` — Comparar rendimiento en tiempo de ejecución.
4. **Lint**: `algo-alchemist lint <file>` — Asegurar cumplimiento de estilo de código (black/flake8 o eslint/prettier).
5. **Verificación de Tipos**: `mypy <file>` o `tsc <file>` — No se introdujeron errores de tipo.
6. **Revisión Manual**: Inspeccionar el diff generado para legibilidad y mantenibilidad.

## Solución de Problemas

**Problema**: La optimización rompe las pruebas existentes.
- **Solución**: `algo-alchemist undo <file>` para revertir. Ajusta manualmente o usa el flag `--preserve-api` para forzar estabilidad de interfaz.

**Problema**: La complejidad no mejoró como se esperaba.
- **Solución**: El algoritmo puede tener un límite inferior inherente. Usa `algo-alchemist suggest` para ver paradigmas alternativos. Considera si las restricciones del problema permiten una mejor complejidad.

**Problema**: El uso de memoria aumentó dramáticamente.
- **Solución**: Revisa las elecciones de estructuras de datos. `algo-alchemist datastruct <file> --optimize memory` para explorar alternativas que ahorren espacio. Intercambia tiempo por espacio si es necesario.

**Problema**: Desbordamiento de pila después de conversión iterativa.
- **Solución**: Revisión manual de la gestión de pila. Asegura una condición de terminación adecuada y que todas las rutas estén cubiertas.

**Problema**: Comando `algo-alchemist` no encontrado.
- **Solución**: Asegúrate de que `~/.local/bin` esté en el PATH o usa `python -m algo_alchemist`. Instala dependencias: `pip install -r requirements.txt`.

**Problema**: El perfilado se cuelga o agota el tiempo.
- **Solución**: Establece `ALGO_TIMEOUT` a un valor mayor (por ejemplo, `export ALGO_TIMEOUT=120`). Verifica el tamaño de los datos de entrada. Usa `--runs 1` para una sola ejecución.

**Problema**: La verificación de tipos falla después de la optimización.
- **Solución**: Algunas transformaciones (como añadir lru_cache) necesitan importación. `algo-alchemist` debería añadir importaciones automáticamente; si no, añade manualmente `from functools import lru_cache`.

## Dependencias

**Requeridas:**
- Python 3.8+ with ast, astunparse, mypy, pytest, timeit, networkx
- Para JavaScript/TypeScript: Node.js 14+, eslint, jest, typescript

**Opcionales (para perfilado):**
- py-spy (perfilador por muestreo, funciona en código de producción)
- memory_profiler (uso de memoria línea por línea)
- pyperf (benchmarking riguroso)

## Configuración

Crea `~/.config/algo-alchemist/config.yaml`:
```yaml
cache_dir: "~/.cache/algo-alchemist"
default_timeout: 30
max_iterations: 10
auto_undo_on_failure: true
languages:
  python:
    style: black
    typecheck: mypy
    test_framework: pytest
  javascript:
    style: prettier
    typecheck: eslint
    test_framework: jest
```

## Notas

- El historial de transformaciones se almacena en `.algo-history.json` (agrega este archivo a .gitignore si compartes).
- Siempre haz commit antes de ejecutar `algo-alchemist optimize` para tener una línea base limpia.
- Usa `--dry-run` con `apply` para previsualizar cambios.
- Para bases de código grandes, optimiza una función a la vez (`--function <name>`) para reducir el riesgo.
- La herramienta respeta las importaciones y dependencias existentes; no añade librerías externas sin aprobación explícita del usuario.