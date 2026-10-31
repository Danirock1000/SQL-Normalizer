# Arquitectura

## Flujo de procesamiento

```
.sql subido
  → lectura        Codificación, separación por GO, clasificación de sentencias
  → comprobación   Sintaxis, estructura, suficiencia de información → veredicto
  → corrección     Sintaxis y diseño                                → corregido.sql
  → comprobación   Segunda pasada sobre el archivo corregido
  → dependencias   DF desde restricciones, datos y heurísticas
  → normalización  Diagnóstico de FN y descomposición (1FN → 2FN → 3FN)
  → generación     normalizado.sql con migración de datos
  → validación     Ejecución en SQL Server (base temporal)
  → diagrama       Texto Mermaid del esquema final
  → respuesta      Veredicto + diagrama + descargas
```

## Modelo intermedio común

Todos los módulos leen y escriben la misma estructura. Debe definirse **antes** de repartir el trabajo.

```
Esquema
 ├─ tablas[]       nombre, columnas[], pk, fks[], uniques[]
 │                   columna → nombre, tipo, nullable, identity
 ├─ datos          filas por tabla (si existen INSERT)
 ├─ dfs[]          determinante → dependiente, origen: restricción | datos | heurística
 ├─ conservados[]  vistas/procedimientos/funciones/triggers + tablas y columnas que usan
 └─ hallazgos[]    código, severidad, mensaje, línea, acción tomada
```

La lista de **hallazgos** es la base del futuro reporte: cada módulo registra lo que detecta o corrige.

## Fuentes de dependencias funcionales (sin IA)

1. **Restricciones declaradas**: PK y UNIQUE determinan la tabla; las FK indican relaciones.
2. **Datos**: verificación con pandas (`X → A` si `groupby(X)[A].nunique().max() == 1`), lados izquierdos de 1–2 columnas. Solo se acepta si el determinante tiene valores repetidos y hay un mínimo de filas configurable.
3. **Heurísticas de nombres**: prefijos (`cliente_id` → `cliente_*`), grupos repetidos (`telefono1..N`), dependencias parciales sobre partes de una PK compuesta.
4. **Valores multivaluados** en los datos (listas separadas por `,` o `;`) para la 1FN.

## Puntos delicados de SQL Server / SSMS

- `GO` no es T-SQL: separar lotes antes de usar sqlglot o pyodbc.
- Scripts de SSMS frecuentemente en **UTF-16 con BOM**.
- Opciones de almacenamiento (`WITH (PAD_INDEX ...) ON [PRIMARY]`, `TEXTIMAGE_ON`) se limpian antes del análisis.
- `INSERT [dbo].[T] (...) VALUES` sin `INTO`, cadenas `N'...'`, `CAST(N'...' AS Date)`.

## Reparto del trabajo

| Persona | Responsabilidad |
|---|---|
| A | `lectura/`, `correccion/` y la parte de sintaxis de `comprobacion/` (incluida la conexión a SQL Server) |
| B | `dependencias/`, `normalizacion/` y la parte de suficiencia de información de `comprobacion/` |
| C | `modelo/`, `generacion/`, `diagrama/`, `web/` y los casos de prueba |

## Plan de 2 semanas

- **Día 1**: todo el equipo define el modelo intermedio y los casos de prueba con su resultado esperado.
- **Días 2–7**: cada persona desarrolla su módulo, con datos simulados donde dependa de otro.
- **Días 8–10**: integración.
- **Días 11–14**: pruebas con todos los casos, ajustes visuales y ensayo de la exposición.
