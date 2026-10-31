# Casos de prueba — módulo de comprobación

Cada caso es un archivo `.sql` en esta carpeta con el nombre indicado. Los archivos se conservan byte a byte (ver `.gitattributes`).

| # | Archivo | Contenido | Veredicto esperado |
|---|---|---|---|
| 01 | `01_ya_3fn.sql` | Script limpio, ya en 3FN | `YA_NORMALIZADO` — `corregido.sql` + diagrama |
| 02 | `02_transitiva_con_datos.sql` | Dependencia transitiva, con `INSERT` | `APTO` — techo 3FN, confianza alta |
| 03 | `03_parcial_con_datos.sql` | PK compuesta con dependencia parcial, con `INSERT` | `APTO` |
| 04 | `04_grupo_repetido.sql` | Columnas `telefono1`, `telefono2`, `telefono3` | `APTO` — FN actual 0FN |
| 05 | `05_transitiva_sin_datos.sql` | Igual que el 02, sin `INSERT` | `APTO` — confianza media |
| 06 | `06_sin_pk_sin_patron.sql` | Tablas sin PK y nombres sin patrón, sin datos | `APTO_PARCIAL` |
| 07 | `07_errores_sintaxis.sql` | Coma sobrante, `PRIMAY KEY`, `;` faltante | `APTO_CON_CORRECCIONES` |
| 08 | `08_export_ssms_utf16.sql` | Script de SSMS: `GO`, `[dbo]`, `ON [PRIMARY]`, UTF-16 | `APTO` |
| 09 | `09_sintaxis_mysql.sql` | `AUTO_INCREMENT`, comillas invertidas | `APTO_CON_CORRECCIONES` |
| 10 | `10_solo_vistas_procs.sql` | Solo vistas y procedimientos | `NO_APTO` (sin `CREATE TABLE`) |
| 11 | `11_vacio.sql` | Archivo vacío (y variante binaria) | `NO_APTO` |
| 12 | `12_error_irreparable.sql` | Texto sin sentido dentro de un `CREATE TABLE` | `NO_APTO` |
| 13 | `13_insert_columnas_distintas.sql` | `INSERT` con número de valores distinto al de columnas | `APTO_CON_CORRECCIONES` o advertencia |
| 14 | `14_insert_tabla_inexistente.sql` | `CREATE TABLE` válidos + `INSERT` a tabla inexistente | `APTO` — hallazgo `W010`, `INSERT` comentado |
| 15 | `15_vista_afectada.sql` | Tabla a descomponer + vista que usa una columna que se mueve | `APTO` — vista comentada con advertencia |

## Pendiente

Escribir los 15 archivos `.sql` (día 1, todo el equipo).
