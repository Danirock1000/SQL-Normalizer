# Registro de decisiones

| # | Tema | Decisión |
|---|---|---|
| 1 | Dialecto | Solo SQL Server (T-SQL) |
| 2 | IA | No se usa IA en el procesamiento de archivos |
| 3 | Lenguaje | Python 3.12 |
| 4 | Contenido de los archivos | Desconocido: el motor debe funcionar con y sin `INSERT` |
| 5 | Correcciones | Sintaxis y diseño; ante varias opciones, la más probable |
| 6 | Migración de datos | Sí, siempre que el archivo traiga `INSERT` |
| 7 | Algoritmo | Libre; se usa síntesis de Bernstein para 3FN y separación propia para 1FN |
| 8 | Validación | SQL Server (`SET PARSEONLY ON` y ejecución en base temporal); plan B: sqlglot |
| 9 | Diagrama E-R | Solo del resultado final (del corregido si ya estaba en 3FN) |
| 10 | Archivo ya en 3FN | Devolver `corregido.sql` y el diagrama |
| 11 | Sin `CREATE TABLE` | `NO_APTO` |
| 12 | `INSERT` a tabla inexistente | Hallazgo `W010`, se excluye y se conserva comentado |
| 13 | Vistas y procedimientos | Se conservan; los afectados por la descomposición quedan comentados con advertencia |
| 14 | Despliegue | Ejecución local para la exposición |
| 15 | Reporte explicativo | No por ahora; la lista de hallazgos deja la puerta abierta |
