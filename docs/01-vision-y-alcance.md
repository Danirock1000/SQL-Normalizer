# Visión y alcance

## Objetivo

Una aplicación web local que recibe un archivo `.sql` de SQL Server, lo procesa sin intervención del usuario y devuelve el archivo corregido, el archivo normalizado y el diagrama E-R del resultado.

## Dentro del alcance

- Dialecto: **solo SQL Server (T-SQL)**, incluidos scripts exportados desde SSMS.
- Detección de si el archivo es apto para normalizar, con veredicto y motivos.
- Corrección de errores de **sintaxis y de diseño**; si hay varias correcciones posibles, se aplica la más probable.
- Normalización hasta 3FN; si la información no alcanza, hasta 2FN o 1FN.
- **Migración de datos** a las nuevas tablas cuando el archivo trae `INSERT`.
- Diagrama E-R del resultado mostrado en la app web.
- Ejecución local (no se publica en internet).

## Fuera del alcance

- Uso de IA / LLM en el procesamiento.
- Otros dialectos de SQL (los fragmentos de MySQL u otros se tratan como errores corregibles).
- Normalización de vistas, procedimientos, funciones y triggers (se conservan, ver decisiones).
- Formas normales superiores a 3FN (FNBC, 4FN, 5FN).

## Entradas y salidas

| Caso | `corregido.sql` | `normalizado.sql` | Diagrama E-R |
|---|---|---|---|
| Apto, requiere normalización | ✅ | ✅ (con datos migrados si hay `INSERT`) | Del normalizado |
| Ya está en 3FN | ✅ | — | Del corregido |
| No apto | — | — | — (se muestran los motivos) |

## Abierto a futuro

- Reporte explicativo (`reporte.md` / `reporte.json`) generado a partir de la lista de hallazgos.
- Vistas de compatibilidad para que los objetos conservados sigan funcionando tras la descomposición.
