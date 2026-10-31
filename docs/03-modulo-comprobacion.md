# Módulo de comprobación

Determina, antes de normalizar, si el archivo se puede procesar y hasta qué forma normal.

## Veredictos

| Veredicto | Significado |
|---|---|
| `APTO` | Legible y con información suficiente para intentar 3FN |
| `APTO_CON_CORRECCIONES` | Tiene errores de sintaxis o diseño corregibles; tras corregirlos, se puede normalizar |
| `APTO_PARCIAL` | Se puede procesar, pero algunas tablas solo alcanzan 2FN o 1FN con la información disponible |
| `YA_NORMALIZADO` | Ya está en 3FN: se devuelve `corregido.sql` y el diagrama |
| `NO_APTO` | No se puede procesar |

## Criterios de NO APTO

1. El archivo no se puede leer: vacío, binario o codificación no reconocida.
2. No contiene ningún `CREATE TABLE`.
3. Tiene errores de sintaxis irreparables **dentro de los `CREATE TABLE`**.

Los errores irreparables fuera de los `CREATE TABLE` (por ejemplo, dentro de un procedimiento) no bloquean el archivo: ese objeto se conserva comentado y se registra un hallazgo.

## Etapas

### Etapa 0 — Archivo
- Detección de codificación en orden: UTF-8 (con o sin BOM), UTF-16, Windows-1252.
- Rechazar archivos vacíos o binarios.

### Etapa 1 — Separación y clasificación
- Separar lotes por `GO`, luego sentencias.
- Clasificación:

| Tipo | Acción |
|---|---|
| `CREATE TABLE` | Se procesa |
| `ALTER TABLE ... ADD CONSTRAINT` (PK, FK, UNIQUE) | Se procesa |
| `INSERT` | Se procesa |
| `CREATE DATABASE`, `USE`, `SET ...` | Se conservan sin analizar |
| Vistas, procedimientos, funciones, triggers, `sp_addextendedproperty` | Se conservan; se registran las tablas y columnas que usan |

- Limpiar opciones de almacenamiento de SSMS antes del análisis.

### Etapa 2 — Sintaxis
- Validar cada lote con SQL Server usando `SET PARSEONLY ON` (mensaje oficial y número de línea).
- Plan B configurable: solo sqlglot.
- Clasificar cada error como **corregible** (existe regla en el catálogo) o **no corregible**.

### Etapa 3 — Estructura
- Cada tabla tiene al menos una columna, sin nombres duplicados y con tipos T-SQL válidos.
- Cada `INSERT` apunta a una tabla existente y el número de valores coincide con el de columnas.
  - `INSERT` hacia una tabla inexistente: hallazgo `W010`, se excluye del procesamiento y se conserva comentado.
- Cada FK apunta a una tabla y columna existentes.

### Etapa 4 — Suficiencia de información (techo alcanzable)

| Para asegurar | Hace falta | Fuentes |
|---|---|---|
| 1FN | Clave identificable y valores atómicos | PK/UNIQUE declarados o columna `*_id`; datos o nombres para detectar grupos repetidos y listas |
| 2FN | Saber si hay dependencias parciales de una clave compuesta | Datos o heurística de nombres |
| 3FN | DF entre columnas no clave | Datos con filas suficientes o heurística de prefijos |

Reglas:
- Tabla de 1–2 columnas: ya está en 3FN por definición.
- DF obtenida de datos: solo es fiable si el determinante tiene valores repetidos y hay un mínimo de filas (configurable, por defecto 5).
- Sin datos: el techo depende de las heurísticas y queda con confianza **media**.
- Sin datos ni patrones reconocibles: solo 1FN si hay clave → `APTO_PARCIAL`.

## Salida

```
Diagnóstico
 ├─ veredicto_global
 ├─ tablas[]
 │    ├─ nombre, columnas, filas_de_datos
 │    ├─ clave_detectada + origen (declarada | unique | heurística | sustituta)
 │    ├─ fuentes_de_df (restricciones | datos | heurística)
 │    ├─ fn_actual_preliminar (0FN | 1FN | 2FN | 3FN)
 │    └─ techo_alcanzable + confianza (alta | media | baja)
 └─ hallazgos[]
      └─ código, severidad (ERROR | ADVERTENCIA | INFO), mensaje, línea, corregible
```

Códigos de hallazgo: `E###` errores, `W###` advertencias, `I###` información. El catálogo completo está pendiente de definir.

## Flujo

comprobación → corrección → **segunda comprobación** → normalización.
