# 📰 Observatorio de Conflictividad Laboral en Córdoba
### Bilingual Documentation / Documentación bilingüe

> **Proyecto / Project:** Observatorio de Conflictividad Laboral — Provincia de Córdoba
> **Herramientas / Tools:** Python · Pandas · Matplotlib · Seaborn · NetworkX · Google Colab
> **Corpus:** Base de notas periodísticas codificadas de diarios de Córdoba (2012–2026)
> **Última actualización / Last updated:** Agosto 2026 / August 2026

---

## 🌐 Idioma / Language

- [🇦🇷 Versión en Español](#versión-en-español)
- [🇬🇧 English Version](#english-version)

---

# Versión en Español

## Índice

- [1. Estructura general del proyecto](#1-estructura-general-del-proyecto)
- [2. Esquema de columnas de la base](#2-esquema-de-columnas-de-la-base)
- [Fase 0 — Configuración](#fase-0--configuración)
- [Fase 1 — Carga de datos](#fase-1--carga-de-datos)
- [Fase 2 — Control de calidad del corpus](#fase-2--control-de-calidad-del-corpus)
- [Fase 3 — Análisis descriptivos](#fase-3--análisis-descriptivos)
- [Fase 4 — Análisis temporales avanzados](#fase-4--análisis-temporales-avanzados)
- [Fase 5 — Perfil editorial, redes y actores](#fase-5--perfil-editorial-redes-y-actores)
- [Fase 6 — Motor gráfico unificado](#fase-6--motor-gráfico-unificado)
- [Fase 7 — Carga de bases adicionales](#fase-7--carga-de-bases-adicionales)
- [Referencias metodológicas](#referencias-metodológicas)

---

## 1. Estructura general del proyecto

El proyecto se organiza como un pipeline modular de ocho fases. Las fases 0 a 2 son obligatorias siempre; las fases 3 a 7 pueden ejecutarse en cualquier orden una vez que el DataFrame limpio (`df_limpio`) está disponible.

```
┌──────────────────────────────────────────────────────────────────┐
│  Fase 0 — Configuración               Instalación + imports      │
│  Fase 1 — Carga de datos              Excel (Google Drive)       │
│  Fase 2 — Control de calidad          Excepciones + duplicados   │
│  Fase 3 — Análisis descriptivos       Frecuencias, heatmap, etc. │
│  Fase 4 — Análisis temporales         Picos, períodos, evolución │
│  Fase 5 — Perfil editorial y redes    Shannon, co-ocurrencia     │
│  Fase 6 — Motor gráfico unificado     19 tipos de análisis       │
│  Fase 7 — Bases adicionales           Otros años / períodos      │
└──────────────────────────────────────────────────────────────────┘
```

A diferencia de un pipeline de scraping en vivo, este proyecto parte de **datos ya codificados manualmente** por el equipo de investigación: cada fila de la base representa una nota periodística ya clasificada según dinámica conflictual, actores, formato de acción, demanda y participación. Esto elimina la necesidad de clasificadores automáticos por regex (como en un proyecto de scraping puro) y desplaza el foco metodológico hacia el **control de calidad del corpus codificado** y el **análisis descriptivo/temporal** de las categorías ya asignadas.

La **unidad de análisis** es la nota periodística codificada, identificada por su `Título` y su `Link`. El corpus cubre múltiples diarios de la provincia de Córdoba y un período extenso (2012–2026, según la base cargada), lo que permite tanto cortes sincrónicos (perfil por diario en un año) como diacrónicos (evolución de dinámicas o actores a través del tiempo).

---

## 2. Esquema de columnas de la base

| Columna | Tipo | Descripción |
|---|---|---|
| `Link` | `str` | URL de la nota original |
| `Palabras` | `str` | Palabra(s) clave que motivó la inclusión de la nota en el corpus |
| `Título` | `str` | Titular de la noticia (unidad de análisis) |
| `Contenido` | `str` | Cuerpo de la nota (cuando está disponible) |
| `Fecha` | `datetime` | Fecha de publicación |
| `Mes` / `Año` | `str` / `int` | Mes y año derivados de `Fecha` |
| `Diario` | `str` | Medio que publicó la nota |
| `Pertenencia Sectorial` | `str` | Público / Privado / Mixto / Multisectorial / etc. |
| `Código DC` / `Dinámica Conflictual` | `str` | Clasificación temática del conflicto |
| `Sector` / `Sub sector` | `str` | Rama de actividad económica |
| `Organización detallada` / `Organización Agregada` | `str` | Actor(es) laboral(es) mencionados |
| `Departamento` | `str` | Departamento de la provincia donde ocurre el hecho |
| `Antagonista` / `Actor Estatal` / `Nivel de Gobierno` | `str` | Contraparte del conflicto |
| `Iniciativa Estado` / `Resp. Estado` | `str` | Rol del Estado en el conflicto |
| `Participación` | `str` | Nivel de involucramiento de lxs trabajadorxs (base, conducción, ambos) |
| `Demanda princ. Abierta` / `Demanda principal agrupada` | `str` | Reclamo central de la acción |
| `Formato principal` / `Formato agregado` / `Tipo Formato AC` | `str` | Tipo de acción (directa/indirecta, paro, movilización, etc.) |

> **Nota:** los nombres de columna pueden variar levemente entre archivos de distintos años (espacios sobrantes, mayúsculas). La Fase 1 y la Fase 7 incluyen normalización automática para evitar errores por este motivo.

---

## Fase 0 — Configuración

### Instalación de dependencias

**Descripción.** Instala `networkx` y `wordcloud`, las únicas librerías no incluidas por defecto en Colab que usa el proyecto (`pandas`, `numpy`, `matplotlib` y `seaborn` ya vienen preinstaladas).

**Uso.** Se ejecuta una sola vez por sesión de Colab, al reiniciar el entorno debe correrse de nuevo.

**Interpretación.** No produce salida analítica. Un error de instalación bloqueará las celdas de la Fase 5 que dependen de `networkx` (red de co-ocurrencia) y de la Fase 3 que depende de `wordcloud` (nube de palabras).

**Justificación.** Explicitar dependencias en una celda separada es una práctica estándar de reproducibilidad (Knuth, 1984): permite a cualquier colaborador replicar el entorno de análisis sin conocimiento previo del proyecto.

---

## Fase 1 — Carga de datos

**Descripción.** Monta Google Drive, lee el archivo Excel de la base (`pd.read_excel`) especificando la hoja correcta, limpia espacios sobrantes en los nombres de columna con `str.strip()`, y parsea `Fecha` a formato `datetime` con `dayfirst=True`.

**Uso.** Es la celda de entrada obligatoria del pipeline. Requiere ajustar `RUTA_ARCHIVO` y `NOMBRE_HOJA` según el archivo real en el Google Drive del investigador.

**Interpretación.** `df.info()` confirma que `Fecha` quedó tipada como `datetime64` (no como texto) y muestra la cantidad de valores nulos por columna, lo que permite detectar de entrada columnas con codificación incompleta.

**Justificación.** El formato `.xlsx` fue elegido porque preserva estructura de hojas múltiples (útil cuando la base separa datos por año o por tipo de análisis, como se vio en la base original con las hojas "Púb y Priv", "Participación" y "Base"), y es directamente auditable por el equipo de investigación sin herramientas adicionales.

---

## Fase 2 — Control de calidad del corpus

**Descripción.** Aplica dos mecanismos de limpieza independientes sobre `Título`:

- **A) Excepciones (falsos positivos):** un diccionario de expresiones regulares agrupadas por categoría (`cortes_servicios`, `trabajo_obra`, `paro_medico`, etc.) detecta títulos que activarían un filtro laboral por ambigüedad léxica (ej. "corte de luz" vs. "corte de ruta"), pero que en realidad no corresponden a conflictividad laboral.
- **B) Similitud de Jaccard:** compara pares de títulos publicados por distintos diarios en una ventana de ±1 día, y marca como potencial "mismo evento" a aquellos con una similitud léxica igual o mayor a un umbral configurable (0.6 por defecto).

**Uso.** `limpiar_corpus(df)` se ejecuta una vez, inmediatamente después de la carga, y genera tres salidas: `df_limpio` (corpus depurado de falsos positivos), `df_falsos` (registro de lo descartado, para auditoría) y `df_duplicados` (pares de notas que cubren el mismo evento, que **no se eliminan automáticamente**, quedando a criterio del investigador si se conservan para análisis de agenda comparada entre medios).

**Interpretación.** El resumen impreso por `limpiar_corpus()` indica cuántos falsos positivos se descartaron y por qué excepción, y cuántos pares de duplicados semánticos se detectaron. Un número alto de falsos positivos en una categoría específica sugiere que el corpus original tiene sesgo hacia ese tipo de ambigüedad léxica y que el diccionario de excepciones podría necesitar ajustes adicionales.

**Justificación.** A diferencia de un corpus generado por scraping (donde el filtro de ingreso es un regex de palabras clave sobre texto libre), esta base ya fue codificada manualmente por el equipo de investigación. El control de calidad aquí cumple una función distinta: auditar la consistencia del corpus codificado (fechas, duplicados entre fuentes, columnas vacías) más que filtrar contenido no laboral, que en principio ya fue descartado en el proceso de codificación manual. El test de calidad pre/post (`test_calidad_corpus()`) cuantifica explícitamente el impacto de la limpieza sobre: registros totales, duplicados exactos y por `Link`, fechas no parseables o fuera de rango (2012–2026), inconsistencias entre `Año` y el año real de `Fecha`, links con formato inválido, y celdas vacías o con placeholders (`"sin datos"`, `"s/d"`) en `Dinámica Conflictual`.

---

## Fase 3 — Análisis descriptivos

**Descripción.** Conjunto de análisis exploratorios básicos sobre `df_limpio`: vista general (`info()`), frecuencia por `Diario`, frecuencia por `Dinámica Conflictual` con tabla cruzada Diario × Dinámica, serie temporal simple por fecha, heatmap, nube de palabras extraída de `Título`, y resumen ejecutivo con las métricas centrales del corpus.

**Uso.** Se ejecuta como primer paso analítico tras la limpieza, antes de cualquier análisis avanzado. No requiere parámetros: opera directamente sobre `df_limpio`.

**Interpretación.** Una distribución muy desigual entre diarios (uno con más del 50% del corpus) indica que los análisis comparativos posteriores estarán sesgados hacia ese medio y conviene normalizar por proporción antes de comparar. La categoría (`Dinámica Conflictual`) dominante refleja qué tipo de conflicto tuvo mayor visibilidad mediática en el período — lo que no necesariamente coincide con su frecuencia real en la sociedad, sino con las decisiones editoriales de cobertura.

**Justificación.** El análisis exploratorio inicial (EDA) es un requisito metodológico antes de interpretar cualquier resultado (Tukey, 1977). El volumen diferencial entre fuentes es uno de los sesgos más comunes en análisis de medios comparado (Neuendorf, 2017), y explicitarlo desde el comienzo permite tomar decisiones metodológicas conscientes sobre normalización.

---

## Fase 4 — Análisis temporales avanzados

**Descripción.** Incluye seis análisis: (1) evolución de notas por día/semana/mes con media móvil de 7 días y composición mensual por dinámica conflictual (área apilada); (2) comparación de métricas entre dos períodos definidos por una `FECHA_CORTE` configurable; (3) detección automática de picos de conflictividad (días con notas > media + N·σ); (4) evolución mensual de las organizaciones más mencionadas, con heatmap organización × mes; (5) clasificación y evolución de acciones directas vs. indirectas a partir de `Formato agregado`.

**Uso.** Todas las funciones aceptan `año_inicio` y `año_fin` como parámetros, lo que permite acotar el análisis a cualquier ventana temporal de la base (por ejemplo, comparar 2023–2024 vs. 2025–2026). La comparación de períodos requiere además indicar la `FECHA_CORTE` correspondiente a un evento de interés (cambio de gestión, paro general, reforma laboral).

**Interpretación.** Un pico de conflictividad con alta concentración en una sola `Dinámica Conflictual` y un solo `Diario` sugiere un evento sectorial cubierto en profundidad por ese medio; un pico distribuido entre múltiples dinámicas y diarios sugiere un evento de impacto amplio (paro general, medida de política económica). En la comparación de períodos, un aumento significativo de "notas por día" en el período posterior a la fecha de corte indica que el evento de referencia intensificó la cobertura de conflictividad laboral.

**Justificación.** El diseño pre-post es el más simple de los diseños cuasi-experimentales en ciencias sociales: no permite establecer causalidad, pero sí documentar covariación temporal entre un evento y la cobertura mediática, pregunta central en estudios de agenda-setting en contextos de cambio político (McCombs & Shaw, 1972). La detección estadística de picos evita depender exclusivamente del conocimiento previo del investigador sobre el período (Tarrow, 2011).

---

## Fase 5 — Perfil editorial, redes y actores

**Descripción.** Cuatro análisis que caracterizan la cobertura por diario y por actor: (1) perfil temático por diario (distribución porcentual de `Dinámica Conflictual` normalizada por fila, visualizada como barras apiladas al 100%); (2) red de co-ocurrencia de palabras extraídas de `Título` usando NetworkX, con umbral de frecuencia configurable; (3) índice de diversidad temática por diario mediante entropía de Shannon sobre `Dinámica Conflictual`; (4) ranking de organizaciones más mencionadas (`Organización detallada` u `Organización Agregada`), con conteo de en cuántos diarios distintos aparece cada una.

**Uso.** El perfil editorial y la diversidad Shannon operan sobre la misma tabla cruzada Diario × Dinámica Conflictual, por lo que conviene interpretarlos en conjunto: el perfil muestra *qué* cubre cada diario, la entropía resume *cuán concentrada o dispersa* es esa cobertura en un único número comparable. La red de co-ocurrencia y el ranking de organizaciones aceptan `año_inicio`/`año_fin` para acotar el período de análisis.

**Interpretación.** Un diario con entropía cercana al máximo teórico (`log₂(n_dinámicas)`) cubre todos los tipos de conflicto con frecuencias similares; un diario con entropía baja concentra su cobertura en pocas dinámicas conflictuales. En la red de co-ocurrencia, un nodo con alta centralidad de grado es un término que aparece en contextos variados dentro de los títulos — es decir, es conceptualmente central en el discurso periodístico del corpus.

**Justificación.** La normalización por fila en el perfil editorial es metodológicamente necesaria para eliminar el efecto tamaño entre diarios (Breed, 1955). La entropía de Shannon es el indicador estándar de diversidad en teoría de la información (Shannon, 1948), con ventajas sobre medidas más simples como el índice de Herfindahl porque es sensible tanto a la distribución de frecuencias como al número de categorías posibles. El análisis de redes semánticas permite identificar la estructura implícita del discurso periodístico sobre conflictividad laboral (Manning & Schütze, 1999).

---

## Fase 6 — Motor gráfico unificado

**Descripción.** Un sistema de dos capas: (1) la función genérica `graficar()`, que construye tablas dinámicas Período × Categoría a partir de cualquier columna de la base y las visualiza como barras apiladas (verticales u horizontales), líneas, o barras apiladas al 100%; (2) siete funciones especializadas para los análisis que no se reducen a una simple tabla dinámica (red de co-ocurrencia, diversidad Shannon, organizaciones, evolución de organizaciones, picos, comparación de períodos, directa vs. indirecta). Ambas capas se integran en `menu_graficos()`, un menú interactivo de 19 opciones que reproduce los gráficos de referencia del proyecto y agrega los análisis nuevos, todos bajo una misma interfaz de parámetros.

**Uso.** Se ejecuta `menu_graficos(df_limpio)` y el sistema solicita, por consola: qué análisis generar, la granularidad temporal (año/trimestre/mes), el rango de años, y —según el análisis elegido— parámetros adicionales (top N de categorías, tipo de valores absolutos/porcentuales, umbral de co-ocurrencia, fecha de corte). Puede volver a ejecutarse tantas veces como gráficos distintos se necesiten en una misma sesión.

**Interpretación.** Cada gráfico generado corresponde exactamente a una configuración reproducible de filtro + columna + período, lo que facilita documentar en un informe metodológico exactamente qué subconjunto de datos originó cada visualización — un requisito de transparencia particularmente relevante cuando se comparan resultados entre distintos cortes temporales del mismo corpus.

**Justificación.** Centralizar la lógica de graficado en un único motor parametrizable, en lugar de escribir una celda de código distinta por cada gráfico, reduce la duplicación y minimiza el riesgo de inconsistencias entre visualizaciones que deberían ser comparables entre sí (por ejemplo, dos gráficos de la misma dinámica conflictual en distintos períodos, generados con lógicas de filtrado sutilmente distintas).

---

## Fase 7 — Carga de bases adicionales

**Descripción.** Permite incorporar archivos `.xlsx` correspondientes a otros años o períodos no cubiertos por la base principal. Normaliza los nombres de columna contra un diccionario de variantes conocidas (`MAPA_COLUMNAS_PROYECTO`), verifica la presencia de columnas mínimas (`Título`, `Fecha`, `Diario`), parsea fechas, y concatena el resultado con la base original, eliminando duplicados exactos por `Título` + `Diario` + `Fecha`.

**Uso.** Se ejecuta después de la Fase 1 y antes de la Fase 2, para que el corpus combinado completo pase por el control de calidad de una sola vez. Soporta la carga de múltiples archivos en una misma ejecución, y pregunta interactivamente qué hoja usar cuando un archivo tiene más de una.

**Interpretación.** El log de la celda indica cuántas filas se incorporaron desde cada archivo adicional y cuántos duplicados se eliminaron al combinar con la base original — un solapamiento alto entre archivos sugiere que los períodos de cobertura se superponen y conviene revisar los rangos de fecha de cada fuente antes de dar por buena la unificación.

**Justificación.** En proyectos de investigación longitudinales, es común que la base de datos crezca de forma incremental (un archivo por año o por tanda de codificación). Automatizar la normalización de columnas evita que pequeñas inconsistencias de nomenclatura (mayúsculas, espacios, variantes de nombre) generen columnas duplicadas o pérdida silenciosa de datos al concatenar.

---

## Referencias metodológicas

- Breed, W. (1955). Social control in the newsroom: A functional analysis. *Social Forces*, 33(4), 326–335.
- Knuth, D. E. (1984). Literate programming. *The Computer Journal*, 27(2), 97–111.
- Manning, C. D., & Schütze, H. (1999). *Foundations of Statistical Natural Language Processing*. MIT Press.
- McCombs, M., & Shaw, D. (1972). The agenda-setting function of mass media. *Public Opinion Quarterly*, 36(2), 176–187.
- Neuendorf, K. A. (2017). *The Content Analysis Guidebook* (2nd ed.). SAGE.
- Shannon, C. E. (1948). A mathematical theory of communication. *Bell System Technical Journal*, 27(3), 379–423.
- Tarrow, S. (2011). *Power in Movement: Social Movements and Contentious Politics* (3rd ed.). Cambridge University Press.
- Tukey, J. W. (1977). *Exploratory Data Analysis*. Addison-Wesley.

---
---

# English Version

## Table of Contents

- [1. Overall project structure](#1-overall-project-structure)
- [2. Database column schema](#2-database-column-schema)
- [Phase 0 — Setup](#phase-0--setup)
- [Phase 1 — Data loading](#phase-1--data-loading)
- [Phase 2 — Corpus quality control](#phase-2--corpus-quality-control)
- [Phase 3 — Descriptive analysis](#phase-3--descriptive-analysis)
- [Phase 4 — Advanced temporal analysis](#phase-4--advanced-temporal-analysis)
- [Phase 5 — Editorial profile, networks and actors](#phase-5--editorial-profile-networks-and-actors)
- [Phase 6 — Unified graphing engine](#phase-6--unified-graphing-engine)
- [Phase 7 — Loading additional datasets](#phase-7--loading-additional-datasets)
- [Methodological references](#methodological-references)

---

## 1. Overall project structure

The project is organized as a modular eight-phase pipeline. Phases 0 through 2 are always mandatory; phases 3 through 7 can be run in any order once the cleaned DataFrame (`df_limpio`) is available.

```
┌──────────────────────────────────────────────────────────────────┐
│  Phase 0 — Setup                     Install + imports           │
│  Phase 1 — Data loading              Excel (Google Drive)        │
│  Phase 2 — Quality control           Exceptions + duplicates     │
│  Phase 3 — Descriptive analysis      Frequencies, heatmap, etc.  │
│  Phase 4 — Temporal analysis         Peaks, periods, evolution   │
│  Phase 5 — Editorial profile & nets  Shannon, co-occurrence      │
│  Phase 6 — Unified graphing engine   19 analysis types           │
│  Phase 7 — Additional datasets       Other years / periods       │
└──────────────────────────────────────────────────────────────────┘
```

Unlike a live-scraping pipeline, this project starts from **data already manually coded** by the research team: each row represents a news item already classified by conflict dynamic, actors, action format, demand, and workers' participation. This removes the need for regex-based automatic classifiers (as in a pure scraping project) and shifts the methodological focus toward **quality control of the coded corpus** and **descriptive/temporal analysis** of the already-assigned categories.

The **unit of analysis** is the coded news item, identified by its `Título` (headline) and `Link`. The corpus spans multiple newspapers from the province of Córdoba over an extended period (2012–2026, depending on the loaded file), enabling both synchronic cuts (a newspaper's profile within a given year) and diachronic ones (evolution of conflict dynamics or actors over time).

---

## 2. Database column schema

| Column | Type | Description |
|---|---|---|
| `Link` | `str` | URL of the original news item |
| `Palabras` | `str` | Keyword(s) that triggered the item's inclusion in the corpus |
| `Título` | `str` | News headline (unit of analysis) |
| `Contenido` | `str` | Article body (when available) |
| `Fecha` | `datetime` | Publication date |
| `Mes` / `Año` | `str` / `int` | Month and year derived from `Fecha` |
| `Diario` | `str` | Outlet that published the item |
| `Pertenencia Sectorial` | `str` | Public / Private / Mixed / Multisectoral / etc. |
| `Código DC` / `Dinámica Conflictual` | `str` | Thematic classification of the conflict |
| `Sector` / `Sub sector` | `str` | Economic activity branch |
| `Organización detallada` / `Organización Agregada` | `str` | Labor actor(s) mentioned |
| `Departamento` | `str` | Provincial department where the event took place |
| `Antagonista` / `Actor Estatal` / `Nivel de Gobierno` | `str` | Counterpart of the conflict |
| `Iniciativa Estado` / `Resp. Estado` | `str` | State's role in the conflict |
| `Participación` | `str` | Level of worker involvement (rank-and-file, leadership, both) |
| `Demanda princ. Abierta` / `Demanda principal agrupada` | `str` | Central claim of the action |
| `Formato principal` / `Formato agregado` / `Tipo Formato AC` | `str` | Type of action (direct/indirect, strike, mobilization, etc.) |

> **Note:** column names may vary slightly between files from different years (extra spaces, capitalization). Phase 1 and Phase 7 include automatic normalization to prevent errors from this cause.

---

## Phase 0 — Setup

### Installing dependencies

**Description.** Installs `networkx` and `wordcloud`, the only libraries not pre-installed by default in Colab that the project uses (`pandas`, `numpy`, `matplotlib`, and `seaborn` come pre-installed).

**Usage.** Runs once per Colab session; must be re-run after any runtime restart.

**Interpretation.** Produces no analytical output. An installation failure will block Phase 5 cells that depend on `networkx` (co-occurrence network) and the Phase 3 cell that depends on `wordcloud`.

**Rationale.** Making dependencies explicit in a separate cell is a standard reproducibility practice (Knuth, 1984): it allows any collaborator to replicate the analysis environment without prior knowledge of the project.

---

## Phase 1 — Data loading

**Description.** Mounts Google Drive, reads the Excel database file (`pd.read_excel`) specifying the correct sheet, strips extra whitespace from column names with `str.strip()`, and parses `Fecha` into `datetime` format with `dayfirst=True`.

**Usage.** This is the pipeline's mandatory entry cell. Requires adjusting `RUTA_ARCHIVO` and `NOMBRE_HOJA` to match the actual file in the researcher's Google Drive.

**Interpretation.** `df.info()` confirms that `Fecha` was correctly typed as `datetime64` (not text) and shows the count of missing values per column, allowing early detection of incompletely coded columns.

**Rationale.** The `.xlsx` format was chosen because it preserves multi-sheet structure (useful when the source file separates data by year or by analysis type, as seen in the original file with the "Púb y Priv", "Participación", and "Base" sheets), and is directly auditable by the research team without additional tools.

---

## Phase 2 — Corpus quality control

**Description.** Applies two independent cleaning mechanisms on `Título`:

- **A) Exceptions (false positives):** a dictionary of regular expressions grouped by category (`cortes_servicios`, `trabajo_obra`, `paro_medico`, etc.) flags headlines that would otherwise trigger a labor-related filter due to lexical ambiguity (e.g., "power outage" vs. "road blockade"), but that do not actually correspond to labor conflict.
- **B) Jaccard similarity:** compares pairs of headlines published by different outlets within a ±1-day window, flagging as potential "same event" any pair whose lexical similarity meets or exceeds a configurable threshold (0.6 by default).

**Usage.** `limpiar_corpus(df)` runs once, immediately after loading, and produces three outputs: `df_limpio` (corpus cleaned of false positives), `df_falsos` (log of discarded rows, for audit purposes), and `df_duplicados` (pairs of items covering the same event, which are **not automatically removed** — whether to keep them for cross-media agenda analysis is left to the researcher's judgment).

**Interpretation.** The summary printed by `limpiar_corpus()` indicates how many false positives were discarded and by which exception category, and how many semantic duplicate pairs were detected. A high count in a specific exception category suggests the original corpus has a bias toward that type of lexical ambiguity and that the exceptions dictionary may need further tuning.

**Rationale.** Unlike a scraped corpus (where the entry filter is a keyword regex over free text), this database was already manually coded by the research team. Quality control here serves a different purpose: auditing the consistency of the coded corpus (dates, cross-source duplicates, empty columns) rather than filtering out non-labor content, which was in principle already excluded during manual coding. The pre/post quality test (`test_calidad_corpus()`) explicitly quantifies the impact of cleaning on: total records, exact and `Link`-based duplicates, unparseable or out-of-range dates (2012–2026), inconsistencies between `Año` and the actual year in `Fecha`, malformed links, and empty or placeholder cells (`"sin datos"`, `"s/d"`) in `Dinámica Conflictual`.

---

## Phase 3 — Descriptive analysis

**Description.** A set of basic exploratory analyses over `df_limpio`: general overview (`info()`), frequency by `Diario`, frequency by `Dinámica Conflictual` with a Diario × Dinámica cross-tabulation, simple date-based time series, heatmap, word cloud extracted from `Título`, and an executive summary with the corpus's core metrics.

**Usage.** Run as the first analytical step after cleaning, before any advanced analysis. Requires no parameters: operates directly on `df_limpio`.

**Interpretation.** A very uneven distribution across outlets (one accounting for over 50% of the corpus) indicates that subsequent comparative analyses will be skewed toward that outlet, and normalizing by proportion is advisable before comparing. The dominant category (`Dinámica Conflictual`) reflects which type of conflict received the most media visibility in the period — which does not necessarily coincide with its actual frequency in society, but rather with editorial coverage decisions.

**Rationale.** Initial exploratory data analysis (EDA) is a methodological requirement before interpreting any results (Tukey, 1977). Differential volume across sources is one of the most common biases in comparative media analysis (Neuendorf, 2017), and making it explicit from the outset allows conscious methodological decisions about normalization.

---

## Phase 4 — Advanced temporal analysis

**Description.** Includes six analyses: (1) evolution of items per day/week/month with a 7-day moving average and monthly composition by conflict dynamic (stacked area); (2) comparison of metrics between two periods defined by a configurable `FECHA_CORTE`; (3) automatic detection of conflict peaks (days with items > mean + N·σ); (4) monthly evolution of the most-mentioned organizations, with an organization × month heatmap; (5) classification and evolution of direct vs. indirect actions based on `Formato agregado`.

**Usage.** All functions accept `año_inicio` and `año_fin` as parameters, allowing the analysis to be bounded to any time window in the database (e.g., comparing 2023–2024 vs. 2025–2026). Period comparison additionally requires specifying the `FECHA_CORTE` corresponding to an event of interest (change of administration, general strike, labor reform).

**Interpretation.** A conflict peak highly concentrated in a single `Dinámica Conflictual` and a single `Diario` suggests a sectoral event covered in depth by that outlet; a peak spread across multiple dynamics and outlets suggests a broad-impact event (general strike, economic policy measure). In period comparison, a significant increase in "items per day" after the cutoff date indicates that the reference event intensified labor-conflict coverage.

**Rationale.** The pre-post design is the simplest of quasi-experimental designs in social science: it cannot establish causality, but it does document temporal covariation between an event and media coverage — a central question in agenda-setting studies within contexts of political change (McCombs & Shaw, 1972). Statistical peak detection avoids relying exclusively on the researcher's prior knowledge of the period (Tarrow, 2011).

---

## Phase 5 — Editorial profile, networks and actors

**Description.** Four analyses that characterize coverage by outlet and by actor: (1) thematic profile by outlet (percentage distribution of `Dinámica Conflictual` row-normalized, visualized as 100% stacked bars); (2) co-occurrence network of words extracted from `Título` using NetworkX, with a configurable frequency threshold; (3) thematic diversity index by outlet via Shannon entropy over `Dinámica Conflictual`; (4) ranking of the most-mentioned organizations (`Organización detallada` or `Organización Agregada`), counting how many distinct outlets mention each one.

**Usage.** The editorial profile and Shannon diversity operate on the same Diario × Dinámica Conflictual cross-tabulation, so they are best interpreted together: the profile shows *what* each outlet covers, while entropy summarizes *how concentrated or dispersed* that coverage is in a single comparable number. The co-occurrence network and the organization ranking accept `año_inicio`/`año_fin` to bound the analysis period.

**Interpretation.** An outlet with entropy close to the theoretical maximum (`log₂(n_dynamics)`) covers all conflict types with similar frequencies; an outlet with low entropy concentrates its coverage in few conflict dynamics. In the co-occurrence network, a node with high degree centrality is a term that appears in varied contexts within headlines — i.e., it is conceptually central to the corpus's journalistic discourse.

**Rationale.** Row-wise normalization in the editorial profile is methodologically necessary to eliminate the size effect between outlets (Breed, 1955). Shannon entropy is the standard diversity indicator in information theory (Shannon, 1948), with advantages over simpler measures like the Herfindahl index because it is sensitive both to the frequency distribution and to the number of possible categories. Semantic network analysis allows identifying the implicit structure of journalistic discourse on labor conflict (Manning & Schütze, 1999).

---

## Phase 6 — Unified graphing engine

**Description.** A two-layer system: (1) the generic `graficar()` function, which builds Period × Category pivot tables from any column in the database and visualizes them as stacked bars (vertical or horizontal), lines, or 100% stacked bars; (2) seven specialized functions for analyses that don't reduce to a simple pivot table (co-occurrence network, Shannon diversity, organizations, organization evolution, peaks, period comparison, direct vs. indirect). Both layers are integrated into `menu_graficos()`, an interactive 19-option menu that reproduces the project's reference charts and adds the newer analyses, all under a single parameter interface.

**Usage.** Running `menu_graficos(df_limpio)` prompts, via console: which analysis to generate, the temporal granularity (year/quarter/month), the year range, and — depending on the chosen analysis — additional parameters (top N categories, absolute/percentage values, co-occurrence threshold, cutoff date). It can be re-run as many times as needed within a single session to produce different charts.

**Interpretation.** Each generated chart corresponds exactly to a reproducible configuration of filter + column + period, which makes it easy to document in a methodological report exactly which data subset produced each visualization — a transparency requirement particularly relevant when comparing results across different time cuts of the same corpus.

**Rationale.** Centralizing charting logic in a single parameterizable engine, rather than writing a separate code cell per chart, reduces duplication and minimizes the risk of inconsistencies between visualizations that should be comparable to each other (for example, two charts of the same conflict dynamic in different periods, generated with subtly different filtering logic).

---

## Phase 7 — Loading additional datasets

**Description.** Allows incorporating `.xlsx` files corresponding to other years or periods not covered by the main database. Normalizes column names against a dictionary of known variants (`MAPA_COLUMNAS_PROYECTO`), verifies the presence of minimum required columns (`Título`, `Fecha`, `Diario`), parses dates, and concatenates the result with the original database, removing exact duplicates by `Título` + `Diario` + `Fecha`.

**Usage.** Run after Phase 1 and before Phase 2, so the full combined corpus goes through quality control in a single pass. Supports uploading multiple files in one run, and interactively asks which sheet to use when a file has more than one.

**Interpretation.** The cell's log reports how many rows were incorporated from each additional file and how many duplicates were removed when merging with the original database — a high overlap between files suggests that coverage periods intersect and that each source's date ranges should be reviewed before considering the merge final.

**Rationale.** In longitudinal research projects, it is common for the database to grow incrementally (one file per year or per coding batch). Automating column normalization prevents small naming inconsistencies (capitalization, whitespace, name variants) from generating duplicate columns or silent data loss during concatenation.

---

## Methodological references

- Breed, W. (1955). Social control in the newsroom: A functional analysis. *Social Forces*, 33(4), 326–335.
- Knuth, D. E. (1984). Literate programming. *The Computer Journal*, 27(2), 97–111.
- Manning, C. D., & Schütze, H. (1999). *Foundations of Statistical Natural Language Processing*. MIT Press.
- McCombs, M., & Shaw, D. (1972). The agenda-setting function of mass media. *Public Opinion Quarterly*, 36(2), 176–187.
- Neuendorf, K. A. (2017). *The Content Analysis Guidebook* (2nd ed.). SAGE.
- Shannon, C. E. (1948). A mathematical theory of communication. *Bell System Technical Journal*, 27(3), 379–423.
- Tarrow, S. (2011). *Power in Movement: Social Movements and Contentious Politics* (3rd ed.). Cambridge University Press.
- Tukey, J. W. (1977). *Exploratory Data Analysis*. Addison-Wesley.(3), 379–423.
- Stodden, V., Leisch, F., & Peng, R. D. (2014). *Implementing Reproducible Research*. CRC Press.
- Tarrow, S. (2011). *Power in Movement: Social Movements and Contentious Politics* (3rd ed.). Cambridge University Press.
- Tufte, E. R. (1983). *The Visual Display of Quantitative Information*. Graphics Press.
- Tukey, J. W. (1977). *Exploratory Data Analysis*. Addison-Wesley.
