# SQL-Normalizer

Aplicación web que recibe un archivo `.sql` de **SQL Server (T-SQL)** y, sin pedir datos al usuario:

1. Determina si el archivo cumple los requisitos para ser normalizado.
2. Corrige errores de sintaxis y de diseño (eligiendo la corrección más probable).
3. Normaliza el esquema hasta **3FN**, o hasta 2FN / 1FN si la información disponible no alcanza.
4. Migra los datos (`INSERT`) a las nuevas tablas.
5. Devuelve `corregido.sql` y `normalizado.sql`, y muestra el **diagrama entidad-relación** del resultado.

> Proyecto universitario — ejecución local para exposición. No usa IA en el procesamiento.

## Stack

| Capa | Tecnología |
|---|---|
| Lenguaje | Python 3.12 |
| Backend | FastAPI + Uvicorn |
| Análisis de SQL | sqlglot (dialecto `tsql`), sqlparse |
| Validación de sintaxis | SQL Server (`SET PARSEONLY ON`) vía pyodbc — plan B: solo sqlglot |
| Análisis de datos | pandas |
| Normalización | Implementación propia (síntesis de Bernstein + separación para 1FN) |
| Diagrama E-R | Mermaid.js (`erDiagram`), servido localmente |
| Frontend | HTML + CSS + JavaScript simple |
| Pruebas | pytest |

## Estructura

```
SQL-Normalizer/
├── app/
│   ├── modelo/          Modelo intermedio común (Esquema, Tabla, Columna, DF, Hallazgo)
│   ├── lectura/         Etapas 0-1: codificación, separación por GO, clasificación de sentencias
│   ├── comprobacion/    Etapas 2-4: sintaxis, estructura, suficiencia de información → veredicto
│   ├── correccion/      Corrección de sintaxis y diseño
│   ├── dependencias/    Obtención de dependencias funcionales (restricciones, datos, heurísticas)
│   ├── normalizacion/   Diagnóstico de FN y descomposición (1FN, 2FN, 3FN)
│   ├── generacion/      Generación de corregido.sql y normalizado.sql (con migración de datos)
│   ├── diagrama/        Generación del texto Mermaid del diagrama E-R
│   └── web/             FastAPI + página única (subida, veredicto, diagrama, descargas)
├── docs/                Especificaciones y decisiones del proyecto
└── tests/
    └── casos/           Archivos .sql de prueba con su resultado esperado
```

## Documentación

- [Visión y alcance](docs/01-vision-y-alcance.md)
- [Arquitectura](docs/02-arquitectura.md)
- [Módulo de comprobación](docs/03-modulo-comprobacion.md)
- [Registro de decisiones](docs/decisiones.md)

## Puesta en marcha (cuando exista código)

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

## Equipo

Proyecto de 3 personas — plazo de 2 semanas. Ver el reparto en [Arquitectura](docs/02-arquitectura.md#reparto-del-trabajo).
