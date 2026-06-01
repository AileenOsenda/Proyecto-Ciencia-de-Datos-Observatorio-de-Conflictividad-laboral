# 📰 Documentación — Análisis de Conflictividad Laboral en Medios de Córdoba

> **Proyecto:** Monitoreo de medios gráficos de Córdoba  
> **Herramientas:** Python · Pandas · Matplotlib · Seaborn · NetworkX · Google Colab  
> **Corpus:** Seis diarios de Córdoba (La Izquierda Diario, La Voz del Interior, La Voz de San Justo, El Diario de Villa María, Cba24n y Puntal Río Cuarto)  
> **Última actualización:** Mayo 2025

---

## Índice

- [1. Estructura general del proyecto](#1-estructura-general-del-proyecto)
- [2. Flujo de trabajo sin scraping](#2-flujo-de-trabajo-sin-scraping)
- [Fase I — Configuración](#fase-i--configuración)
  - [Celda 1 — Instalación de dependencias](#celda-1--instalación-de-dependencias)
  - [Celda 2 — Importaciones](#celda-2--importaciones)
  - [Celda 3 — Expresiones regulares y categorías](#celda-3--expresiones-regulares-y-categorías)
  - [Celda 4 — Funciones auxiliares](#celda-4--funciones-auxiliares)
- [Fase II — Carga de datos desde Excel](#fase-ii--carga-de-datos-desde-excel)
  - [Celda 7 — Cargar un único archivo Excel](#celda-7--cargar-un-único-archivo-excel)
  - [Celda 7b — Cargar múltiples archivos Excel](#celda-7b--cargar-múltiples-archivos-excel)
  - [Celda 7c — Diagnóstico de columnas](#celda-7c--diagnóstico-de-columnas)
- [Fase III — Análisis estadístico inicial](#fase-iii--análisis-estadístico-inicial)
  - [Celda 8 — Vista general del dataset](#celda-8--vista-general-del-dataset)
  - [Celda 9 — Frecuencia por diario](#celda-9--frecuencia-por-diario)
  - [Celda 10 — Frecuencia por categoría temática](#celda-10--frecuencia-por-categoría-temática)
  - [Celda 11 — Evolución temporal simple](#celda-11--evolución-temporal-simple)
  - [Celda 12 — Análisis de palabras clave individuales](#celda-12--análisis-de-palabras-clave-individuales)
  - [Celda 13 — Intensidad de cobertura](#celda-13--intensidad-de-cobertura)
  - [Celda 14 — Co-ocurrencia de palabras clave](#celda-14--co-ocurrencia-de-palabras-clave)
- [Fase IV — Visualizaciones](#fase-iv--visualizaciones)
  - [Celda 15 — Gráficos de categoría y diario](#celda-15--gráficos-de-categoría-y-diario)
  - [Celda 16 — Serie temporal de noticias por día](#celda-16--serie-temporal-de-noticias-por-día)
  - [Celda 17 — Heatmap diario × categoría](#celda-17--heatmap-diario--categoría)
  - [Celda 18 — Boxplot de intensidad por categoría](#celda-18--boxplot-de-intensidad-por-categoría)
  - [Celda 19 — Nube de palabras clave](#celda-19--nube-de-palabras-clave)
  - [Celda 20 — Resumen ejecutivo](#celda-20--resumen-ejecutivo)
- [Fase V — Análisis avanzados](#fase-v--análisis-avanzados)
  - [Celda 21 — Perfil editorial por diario](#celda-21--perfil-editorial-por-diario)
  - [Celda 22 — Red de co-ocurrencia](#celda-22--red-de-co-ocurrencia)
  - [Celda 23 — Índice de diversidad temática](#celda-23--índice-de-diversidad-temática-entropía-de-shannon)
  - [Celda 24 — Actores sindicales mencionados](#celda-24--actores-sindicales-mencionados)
  - [Celda 25 — Score de relevancia y ranking](#celda-25--score-de-relevancia-y-ranking-de-noticias)
- [Fase VI — Análisis temporal e histórico](#fase-vi--análisis-temporal-e-histórico)
  - [Celda 27 — Noticias vs. tiempo](#celda-27--noticias-vs-tiempo-análisis-temporal-completo)
  - [Celda 28 — Comparación entre períodos](#celda-28--comparación-entre-períodos)
  - [Celda 29 — Detección de picos de conflictividad](#celda-29--detección-de-picos-de-conflictividad)
  - [Celda 30 — Evolución de actores sindicales](#celda-30--evolución-de-actores-sindicales-en-el-tiempo)
  - [Celda 31 — Sectores afectados vs. movilizados](#celda-31--sectores-afectados-vs-sectores-movilizados)
- [Referencias metodológicas](#referencias-metodológicas)

---

## 1. Estructura general del proyecto

El proyecto se organiza como un pipeline modular de seis fases. Las fases I y II son obligatorias siempre. Las fases III a VI pueden ejecutarse en cualquier orden una vez que el DataFrame está cargado.

```
┌─────────────────────────────────────────────────────────────────┐
│  Fase I   — Configuración       Celdas 1, 2, 3, 4               │
│  Fase II  — Carga de datos      Celdas 7 / 7b / 7c              │
│  Fase III — Análisis inicial    Celdas 8 → 14                   │
│  Fase IV  — Visualizaciones     Celdas 15 → 20                  │
│  Fase V   — Análisis avanzados  Celdas 21 → 25                  │
│  Fase VI  — Análisis temporal   Celdas 27 → 31                  │
└─────────────────────────────────────────────────────────────────┘
```

> **Nota:** Las celdas 5 y 6 (scraping y exportación web) son opcionales y solo se usan si se desea recolectar datos nuevos en tiempo real. Para trabajar con archivos Excel ya existentes, se saltean por completo.

La **unidad de análisis** es el titular periodístico. Esta decisión metodológica responde a que el titular condensa el encuadre editorial de la nota: es el elemento que más influye en la percepción del lector y el que los medios cuidan con mayor deliberación. Analizar titulares en lugar del cuerpo completo permite mayor velocidad de procesamiento y comparabilidad entre medios con distintos estilos de escritura.

El esquema de columnas del DataFrame unificado es el siguiente:

| Columna | Tipo | Descripción |
|---|---|---|
| `fecha_scraping` | `datetime` | Fecha de publicación o recolección |
| `diario` | `str` | Identificador del medio |
| `titular` | `str` | Texto del titular de la noticia |
| `url_noticia` | `str` | Enlace a la nota completa |
| `palabras_clave` | `str` | Lista de KW laborales separadas por coma |
| `n_palabras_clave` | `int` | Cantidad de KW únicas en el titular |
| `categoria` | `str` | Categoría temática asignada |
| `longitud_titular` | `int` | Longitud en caracteres del titular |
| `archivo_origen` | `str` | Nombre del Excel fuente (en carga múltiple) |

---

## 2. Flujo de trabajo sin scraping

Si ya se dispone de archivos `.xlsx` históricos, el flujo de ejecución se reduce a:

```
Celda 1 → Celda 2 → Celda 3 → Celda 4 → Celda 7b → Celda 8 → ...
```

Las celdas 5 y 6 se omiten. La celda 7b reemplaza al scraper como punto de entrada de datos. La función `enriquecer_dataframe()` de la celda 4 actúa como capa de compatibilidad: detecta qué columnas calculadas faltan en el Excel y las genera automáticamente, garantizando que todos los análisis posteriores funcionen independientemente del formato de origen.

---

## Fase I — Configuración

### Celda 1 — Instalación de dependencias

**Descripción.** Instala todas las librerías necesarias mediante `pip`. La lista incluye `requests`, `beautifulsoup4`, `pandas`, `matplotlib`, `seaborn`, `wordcloud`, `cloudscraper`, `openpyxl` y `networkx`. Se ejecuta una sola vez por entorno de Colab; al reiniciar el runtime debe correrse nuevamente.

**Uso.** Debe ejecutarse antes que cualquier otra celda. En entornos donde las librerías ya están instaladas (como un ambiente conda local), puede omitirse. `openpyxl` es el motor de lectura/escritura de archivos `.xlsx` para pandas y es indispensable para la carga de datos históricos.

**Interpretación.** No produce salida analítica. Si alguna instalación falla, la celda imprimirá el error y las celdas siguientes que dependan de esa librería lanzarán un `ImportError`. En ese caso, instalar el paquete faltante de forma aislada antes de continuar.

**Justificación.** La explicitación de dependencias en una celda separada es una práctica estándar de reproducibilidad en ciencia de datos. Permite que cualquier colaborador o revisor pueda replicar el entorno sin conocimiento previo del proyecto, siguiendo el principio de *literate programming* (Knuth, 1984).

---

### Celda 2 — Importaciones

**Descripción.** Centraliza todos los `import` del proyecto. Incluye las librerías estándar (`re`, `time`, `random`, `io`, `itertools`) y las de terceros (`pandas`, `numpy`, `matplotlib`, `seaborn`, `BeautifulSoup`, `cloudscraper`, `wordcloud`, `networkx`). También importa `files` de `google.colab` para la carga y descarga de archivos.

**Uso.** Al igual que la celda 1, debe ejecutarse antes que cualquier otra. Si se trabaja sin scraping, los imports de `requests`, `cloudscraper` y `BeautifulSoup` no se utilizarán pero tampoco generan errores si están instalados.

**Interpretación.** No produce salida analítica. Un error en esta celda indica que una librería no está instalada o que hay un conflicto de versiones.

**Justificación.** Agrupar todos los imports al inicio del notebook es una convención de estilo en Python (PEP 8) y facilita la auditoría de dependencias. Evita el antipatrón de importar librerías dentro de funciones o a mitad del notebook, lo que dificulta el rastreo de errores.

---

### Celda 3 — Expresiones regulares y categorías

**Descripción.** Define dos estructuras centrales del proyecto: la expresión regular `PALABRAS_CLAVES` (más de 80 patrones de términos laborales compilados con `re.IGNORECASE`) y el diccionario `CATEGORIAS` (ocho categorías temáticas, cada una con su propio patrón regex). También define el pool `USER_AGENTS` con agentes de navegador reales para rotación anti-bloqueo en el scraper.

**Uso.** Estas estructuras son referenciadas por prácticamente todas las celdas posteriores. `PALABRAS_CLAVES` es el filtro principal que determina qué noticias ingresan al corpus. `CATEGORIAS` es el clasificador temático que asigna cada noticia a una de las ocho categorías. Ambos pueden modificarse para ajustar la sensibilidad del filtro o agregar nuevas categorías sin tocar el resto del código.

**Interpretación.** El diccionario `CATEGORIAS` define la taxonomía temática del proyecto. Las categorías actuales son: `huelga_paro`, `cortes_moviliz`, `sindicatos`, `salarios`, `despidos_cierre`, `economia_popular`, `docentes` y `desempleo`. La categoría `otra` se asigna automáticamente cuando ningún patrón coincide. La asignación es por **primera coincidencia**: si un titular activa dos categorías, se asigna la primera en el orden del diccionario.

**Justificación.** El uso de expresiones regulares como instrumento de filtrado es una técnica de *text mining* apropiada cuando el vocabulario del dominio es acotado y conocido de antemano, como ocurre con la terminología laboral y sindical argentina. A diferencia de modelos de lenguaje, las regex son deterministas, auditables y no requieren datos de entrenamiento, lo que las hace metodológicamente transparentes para investigación en ciencias sociales.

---

### Celda 4 — Funciones auxiliares

**Descripción.** Define las tres funciones que requieren todas las celdas de análisis cuando se trabaja sin scraping:

- `clasificar_noticia(texto)`: aplica el diccionario `CATEGORIAS` sobre un titular y devuelve la primera categoría coincidente o `"otra"`.
- `calcular_n_palabras_clave(texto_kw)`: cuenta los términos no vacíos en la columna `palabras_clave` (separados por coma).
- `enriquecer_dataframe(df)`: función principal de compatibilidad. Detecta qué columnas calculadas faltan en el DataFrame cargado (`palabras_clave`, `categoria`, `n_palabras_clave`, `longitud_titular`) y las genera aplicando las funciones anteriores. Es **idempotente**: si la columna ya existe, no la modifica.

**Uso.** `enriquecer_dataframe(df)` debe llamarse al final de la celda de carga (7 o 7b), antes de cualquier análisis. Las otras dos funciones son auxiliares internas pero pueden llamarse directamente si se necesita recalcular valores en forma puntual.

**Interpretación.** La salida de `enriquecer_dataframe()` incluye un log de qué columnas se generaron y cuáles ya existían, lo que permite verificar que el Excel de origen tiene el formato esperado. Si todas las columnas ya existen, el log indicará que no se realizó ningún cambio.

**Justificación.** La separación entre funciones de scraping y funciones de análisis responde al principio de responsabilidad única (*Single Responsibility Principle*). Las funciones auxiliares de esta celda son las únicas que deben ejecutarse obligatoriamente en el flujo sin scraping, evitando que el usuario deba entender el funcionamiento del extractor web para poder usar el análisis.

---

## Fase II — Carga de datos desde Excel

### Celda 7 — Cargar un único archivo Excel

**Descripción.** Abre el selector de archivos de Google Colab mediante `files.upload()`, lee el archivo `.xlsx` seleccionado con `pd.read_excel(engine="openpyxl")` y normaliza la columna de fecha a tipo `datetime` con `pd.to_datetime(dayfirst=True, errors="coerce")`. Imprime dimensiones, columnas y rango temporal como verificación inmediata.

**Uso.** Indicada cuando se dispone de un único archivo de datos ya consolidado. Si los datos de distintos períodos o diarios están en archivos separados, usar la celda 7b. Después de esta celda, ejecutar `df = enriquecer_dataframe(df)` (definida en celda 4) antes de continuar con los análisis.

**Interpretación.** Si el rango de fechas impreso no coincide con el período esperado, puede haber filas con fechas mal formateadas que `errors="coerce"` convirtió en `NaT`. Verificar con `df["fecha_scraping"].isna().sum()` cuántas filas quedaron sin fecha válida.

**Justificación.** El formato `.xlsx` fue elegido como formato de trabajo en lugar de `.csv` porque preserva tipos de datos (fechas, números) sin necesidad de especificar `dtype` manualmente, es compatible de forma nativa con Excel para revisión manual de los datos, y soporta múltiples hojas para organizar datasets por período o diario.

---

### Celda 7b — Cargar múltiples archivos Excel

**Descripción.** Permite seleccionar varios archivos `.xlsx` simultáneamente (por ejemplo, `marzo.xlsx`, `abril.xlsx`, `mayo.xlsx`). Para cada archivo, usa `io.BytesIO` para leer los bytes subidos sin necesidad de guardarlos en disco. Agrega la columna `archivo_origen` con el nombre del archivo fuente. Al final concatena todos los fragmentos con `pd.concat()` y elimina duplicados exactos basándose en la combinación `titular + diario + fecha_scraping`.

**Uso.** Es la celda de entrada estándar cuando se trabaja con datos históricos distribuidos en múltiples archivos. El proceso de unificación es:

```
marzo.xlsx  →  DataFrame A  ─┐
abril.xlsx  →  DataFrame B  ─┼─ pd.concat() ─→ DataFrame unificado ─→ drop_duplicates() ─→ df
mayo.xlsx   →  DataFrame C  ─┘
```

Después de esta celda, ejecutar `df = enriquecer_dataframe(df)`.

**Interpretación.** La diferencia entre "Total filas" y "Dataset final" en el log indica el nivel de solapamiento entre archivos: noticias que aparecen en más de un Excel con el mismo titular, diario y fecha. Un solapamiento alto (mayor al 20%) sugiere que los archivos se generaron con períodos superpuestos y deben revisarse para evitar contar eventos dobles en los análisis temporales.

**Justificación.** La deduplicación por `titular + diario + fecha` es más precisa que por URL o por título solo, porque el mismo evento puede tener variaciones mínimas en el titular entre ediciones o entre archivos generados en distintos momentos del día. El uso de `io.BytesIO` evita escribir archivos temporales en el disco de Colab, lo que mejora el rendimiento en entornos con almacenamiento limitado.

---

### Celda 7c — Diagnóstico de columnas

**Descripción.** Celda opcional de diagnóstico previo a la carga. Lee solo la primera fila (`nrows=0`) de cada archivo subido para obtener los nombres de columna sin cargar el dataset completo. Imprime la lista de columnas de cada archivo para que el investigador pueda verificar inconsistencias antes de ejecutar la celda 7b.

**Uso.** Recomendada cuando se trabajan con archivos generados en distintos momentos o por distintas personas, donde los nombres de columna pueden variar (`Titulo` vs `titular`, `Fecha` vs `fecha`). Si se detectan diferencias, ajustar el diccionario `MAPA_COLUMNAS` de la celda 26 antes de ejecutar la carga completa.

**Interpretación.** Si todos los archivos muestran las mismas columnas, la carga unificada funcionará sin problemas. Si hay diferencias, el log indicará exactamente qué columnas difieren entre archivos, permitiendo corregir el mapa de normalización de forma quirúrgica.

**Justificación.** En proyectos colaborativos o longitudinales es común que el esquema de datos evolucione con el tiempo. Un diagnóstico rápido antes de la carga evita que errores de columnas propaguen valores nulos silenciosos a lo largo de todo el análisis posterior.

---

## Fase III — Análisis estadístico inicial

### Celda 8 — Vista general del dataset

**Descripción.** Ejecuta una inspección diagnóstica inicial del DataFrame mediante `df.info()` y `df.describe()`. Muestra el tipo de dato de cada columna, la cantidad de valores no nulos y las estadísticas descriptivas básicas (media, desvío estándar, mínimo, máximo y cuartiles) de las variables numéricas `n_palabras_clave` y `longitud_titular`. También imprime un reporte de valores nulos por columna.

**Uso.** Debe ejecutarse siempre como primer paso tras `enriquecer_dataframe()`. Permite detectar columnas con valores faltantes, tipos de datos incorrectos o rangos anómalos antes de realizar cualquier análisis.

**Interpretación.** Si `n_palabras_clave` tiene una media muy baja (cercana a 1), el filtro de palabras clave está siendo muy selectivo o los titulares son muy cortos. Un valor máximo mayor a 10 indica noticias extremadamente densas, probablemente notas de síntesis. La columna `longitud_titular` no debe tener ceros: si los hay, hay registros con titulares vacíos que deben limpiarse antes de continuar.

**Justificación.** En cualquier investigación basada en datos, el análisis exploratorio inicial (*EDA*) es un requisito metodológico antes de interpretar resultados. Omitir este paso puede llevar a conclusiones erróneas si el dataset contiene ruido, duplicados no detectados o valores atípicos que distorsionen los promedios posteriores (Tukey, 1977).

---

### Celda 9 — Frecuencia por diario

**Descripción.** Calcula cuántas noticias aportó cada medio mediante `value_counts()` y expresa ese conteo también como proporción porcentual sobre el total del corpus. Responde a la pregunta: ¿qué diario cubre más la conflictividad laboral en términos absolutos y relativos?

**Uso.** Sirve como línea de base para todos los análisis comparativos entre medios. Antes de interpretar cualquier diferencia entre diarios es indispensable saber si esa diferencia responde a un encuadre editorial real o simplemente a que ese diario aporta más noticias al corpus.

**Interpretación.** Una distribución muy desigual (un diario con más del 50% del corpus) indica que los análisis posteriores estarán sesgados hacia ese medio. En ese caso conviene normalizar por diario antes de comparar. Si un diario esperado aparece con muy pocas noticias, puede indicar que ese medio efectivamente cubre poco el mundo laboral en el período analizado, o que hubo problemas en la recolección de datos.

**Justificación.** El volumen diferencial entre fuentes es uno de los sesgos más comunes en el análisis de medios. Explicitarlo desde el comienzo permite al investigador tomar decisiones metodológicas conscientes: si trabajar con valores absolutos, proporciones, o realizar análisis estratificados por diario. La normalización por fuente es un requisito metodológico estándar en investigación comparada de medios (Neuendorf, 2017).

---

### Celda 10 — Frecuencia por categoría temática

**Descripción.** Aplica `value_counts()` sobre la columna `categoria` para contar cuántas noticias corresponden a cada tipo de conflicto laboral. Complementariamente genera una tabla cruzada `pd.crosstab(diario, categoria)` que cruza ambas dimensiones en simultáneo, produciendo una matriz de frecuencias absolutas.

**Uso.** Permite responder dos preguntas: ¿qué tipo de conflicto laboral tiene más cobertura en general? ¿Y en cada diario particular? La tabla cruzada es el insumo directo para el heatmap de la celda 17 y para el análisis de perfil editorial de la celda 21.

**Interpretación.** La categoría dominante refleja qué tipo de conflicto es más visible mediáticamente en el período analizado, lo que no necesariamente coincide con cuál es más frecuente en la realidad social. Una alta frecuencia de `salarios` puede indicar un período de paritarias; una alta frecuencia de `despidos_cierre` puede coincidir con ajuste económico. La tabla cruzada revela especializaciones editoriales: si un diario concentra el 70% de sus noticias en `economia_popular`, tiene una línea editorial más atenta a ese sector.

**Justificación.** La clasificación temática es la operacionalización del concepto de *agenda mediática*: el conjunto de temas que los medios priorizan en un período dado. Comparar la agenda por diario permite realizar análisis de agenda-setting, marco teórico central en los estudios de comunicación política y social desde McCombs y Shaw (1972).

---

### Celda 11 — Evolución temporal simple

**Descripción.** Agrupa las noticias por fecha con `groupby("fecha_scraping").size()` para obtener una serie temporal diaria. Calcula el promedio diario, la desviación estándar y los días con mayor y menor actividad informativa. Opera sobre el DataFrame completo (histórico + actual).

**Uso.** Es la introducción al análisis temporal. Brinda una primera visión de si la cobertura laboral es estable o fluctúa significativamente en el tiempo. Sus resultados contextualizan todos los análisis de noticias vs. tiempo de las celdas 27 a 29.

**Interpretación.** Un promedio diario estable con poca varianza indica una cobertura rutinaria del mundo laboral. Picos abruptos señalan eventos extraordinarios (paros nacionales, conflictos sectoriales, medidas de gobierno). La diferencia entre el día más activo y el menos activo es un indicador de la volatilidad del fenómeno en el período estudiado.

**Justificación.** La dimensión temporal es inseparable del análisis de conflictividad laboral. Los conflictos no son estáticos: emergen, escalan y se resuelven. Sin una perspectiva temporal no es posible distinguir si un aumento de noticias refleja un conflicto nuevo o simplemente un mayor seguimiento editorial de un conflicto preexistente (Tarrow, 2011).

---

### Celda 12 — Análisis de palabras clave individuales

**Descripción.** Expande la columna `palabras_clave` (listas separadas por coma) en una Serie plana con una entrada por término usando `str.split().explode()`. Calcula las 20 palabras clave más frecuentes en todo el corpus y la riqueza léxica total (cantidad de términos únicos).

**Uso.** Permite identificar cuáles son los conceptos laborales más utilizados en los titulares durante el período. No mide qué temas son más frecuentes (eso lo hace la categoría), sino qué términos exactos predominan dentro del lenguaje periodístico sobre trabajo.

**Interpretación.** Si `trabajo` o `trabajadores` encabeza el ranking, el corpus es semánticamente genérico. Si `paro` o `despidos` están en los primeros puestos, la cobertura está dominada por conflictos activos. Términos como `paritarias` o `aguinaldo` en posiciones altas son indicadores de períodos de negociación salarial. La riqueza léxica mide la diversidad del vocabulario laboral: un valor bajo indica que pocos términos concentran la cobertura.

**Justificación.** El análisis de frecuencia léxica es una técnica estándar de la lingüística de corpus y el análisis crítico del discurso (Fairclough, 1989). En estudios de medios, la frecuencia con que aparece un término es un indicador de su centralidad en la agenda y de su naturalización como forma de nombrar la realidad social.

---

### Celda 13 — Intensidad de cobertura

**Descripción.** Analiza la variable `n_palabras_clave` como proxy de la densidad temática de cada noticia. Calcula su distribución estadística con `describe()` e identifica las noticias de alta densidad (aquellas con 3 o más palabras clave distintas), imprimiéndolas en tabla junto con su diario y categoría.

**Uso.** Las noticias de alta densidad son las más relevantes para el análisis cualitativo posterior: concentran múltiples dimensiones de la conflictividad laboral en un solo titular. Sirven como muestra estratégica para revisión manual o para entrenar modelos de clasificación más finos.

**Interpretación.** Una noticia con 5 palabras clave como `paro`, `sindicato`, `salario`, `huelga` y `CGT` articula múltiples dimensiones del conflicto laboral en un titular, lo que indica mayor profundidad informativa o mayor complejidad del evento cubierto. La distribución esperada es asimétrica hacia la derecha: la mayoría de las noticias tendrán 1–2 palabras clave y unas pocas concentrarán muchas más.

**Justificación.** No todas las noticias que contienen una palabra clave laboral son igualmente relevantes. La densidad de palabras clave es una forma computacionalmente simple de aproximarse a esa distinción sin necesidad de lectura manual de cada nota, aplicando el principio de relevancia diferencial en el análisis de contenido (Krippendorff, 2018).

---

### Celda 14 — Co-ocurrencia de palabras clave

**Descripción.** Construye todos los pares posibles de palabras clave que aparecen juntas en el mismo titular usando `itertools.combinations`. Cuenta la frecuencia de cada par con `Counter` y muestra los 15 pares más frecuentes. Esta versión textual es la base de la red de la celda 22.

**Uso.** Revela qué conceptos laborales tienden a aparecer juntos en los titulares, lo que indica asociaciones semánticas estables en el discurso periodístico. Es un primer paso hacia el análisis de redes sin necesidad de visualización gráfica.

**Interpretación.** Un par muy frecuente como `paro + sindicato` indica que los medios raramente cubren un paro sin mencionar al actor sindical que lo convoca: hay una asociación narrativa consolidada. Un par como `despido + crisis` sugiere un encuadre económico de las reestructuraciones laborales. Pares inesperados o infrecuentes pueden ser puntos de entrada para el análisis cualitativo.

**Justificación.** El análisis de co-ocurrencia es una técnica de lingüística distribucional que parte del principio de que el significado de una palabra se define por las palabras con las que habitualmente aparece (Firth, 1957). En análisis de medios, permite identificar los *frames* con que los periodistas estructuran los eventos laborales.

---

## Fase IV — Visualizaciones

### Celda 15 — Gráficos de categoría y diario

**Descripción.** Genera un panel de dos gráficos de barras: uno horizontal con la frecuencia de noticias por categoría temática (ordenado de mayor a menor) y uno vertical con la frecuencia por diario. Son las visualizaciones directas de los conteos calculados en las celdas 9 y 10.

**Uso.** Son los gráficos de presentación más básicos del proyecto, adecuados para informes o presentaciones como síntesis del corpus. Permiten comunicar de forma inmediata cuál es la categoría dominante y qué medio es el más activo, sin necesidad de leer tablas numéricas.

**Interpretación.** La orientación horizontal del gráfico de categorías facilita la lectura de etiquetas largas y permite ordenar de mayor a menor sin saturar el eje. En el gráfico de diarios, diferencias muy marcadas en la altura de las barras indican que el corpus no está balanceado entre fuentes.

**Justificación.** La visualización de frecuencias es el estándar de comunicación en ciencias sociales computacionales. El uso de barras horizontales para categorías nominales con etiquetas largas sigue las recomendaciones de diseño de Tufte (1983) y Cairo (2016) para maximizar la legibilidad sin sacrificar información.

---

### Celda 16 — Serie temporal de noticias por día

**Descripción.** Grafica la serie diaria de noticias como una línea con área rellena. Superpone una media móvil de 7 días calculada con `rolling(7)` que suaviza el ruido del ciclo semanal (los fines de semana los diarios publican menos).

**Uso.** Es la visualización temporal básica del corpus. Permite detectar si hay períodos de alta actividad informativa sostenida o si los picos son eventos puntuales. La media móvil separa la tendencia del ruido cotidiano.

**Interpretación.** Picos que superan la media móvil indican días de cobertura extraordinaria, probablemente asociados a eventos laborales concretos (un paro nacional, un conflicto sectorial). Períodos prolongados por encima de la media móvil indican una escalada sostenida. Valles pronunciados corresponden típicamente a fines de semana, feriados o períodos vacacionales.

**Justificación.** La media móvil de 7 días es el estándar en el análisis de series temporales con ciclos semanales. Sin ella, el gráfico diario resulta ruidoso y dificulta la lectura de tendencias. Su uso permite distinguir el *ruido* del ciclo semanal de las variaciones realmente significativas (Hyndman & Athanasopoulos, 2021).

---

### Celda 17 — Heatmap diario × categoría

**Descripción.** Genera un mapa de calor con `seaborn.heatmap` donde las filas son los diarios, las columnas son las categorías temáticas y el valor de cada celda es la cantidad de noticias en esa intersección. La paleta `YlOrRd` codifica la magnitud: celdas más oscuras indican mayor concentración de cobertura.

**Uso.** Es la forma más compacta de visualizar la relación entre dos variables categóricas. Permite detectar patrones de especialización (un diario que concentra su cobertura en una o dos categorías) o de dispersión (medios que cubren todas las categorías de forma pareja).

**Interpretación.** Una celda muy oscura en la intersección de un diario y una categoría indica cobertura desproporcionada de ese tema. Las celdas con valor 0 son igualmente significativas: revelan qué temas son ignorados por qué medios. Filas uniformemente claras pueden indicar un diario con baja cobertura laboral en general.

**Justificación.** El heatmap es la visualización canónica para matrices de frecuencias en ciencias sociales. Permite procesar simultáneamente información sobre dos dimensiones categóricas, tarea que sería imposible con series de gráficos separados. Es especialmente útil cuando el número de categorías hace inmanejable una tabla numérica.

---

### Celda 18 — Boxplot de intensidad por categoría

**Descripción.** Grafica la distribución de `n_palabras_clave` para cada categoría temática mediante diagramas de caja. Cada caja muestra la mediana (línea roja), los cuartiles 25 y 75 (bordes de la caja) y los valores atípicos (puntos fuera de los bigotes).

**Uso.** Responde a la pregunta: ¿hay categorías temáticas que concentran titulares más densos en términos laborales? Compara la complejidad informativa entre categorías, lo que no es posible con un simple conteo de frecuencias.

**Interpretación.** Una categoría con mediana alta (por ejemplo, `huelga_paro` con mediana de 4 KW) indica que esas noticias articulan múltiples conceptos laborales en el mismo titular. Una categoría con mediana baja y poca dispersión indica titulares más simples. Los valores atípicos son noticias excepcionalmente densas que merecen revisión cualitativa.

**Justificación.** El boxplot es la herramienta estándar para comparar distribuciones entre grupos cuando no se puede asumir normalidad. En este contexto, `n_palabras_clave` tiene una distribución asimétrica (pocos titulares con muchas KW, muchos con pocas), por lo que la mediana es un estadístico más representativo que la media.

---

### Celda 19 — Nube de palabras clave

**Descripción.** Genera una visualización tipo *wordcloud* donde el tamaño de cada palabra es proporcional a su frecuencia en el corpus completo. Se configura con `collocations=False` para evitar repetición de bigramas y con `colormap="Blues"` para una lectura limpia.

**Uso.** Es principalmente una herramienta de comunicación visual, útil para presentaciones, informes ejecutivos o materiales de divulgación. Transmite en segundos cuál es el vocabulario dominante del corpus sin necesidad de leer tablas de frecuencias.

**Interpretación.** Las palabras más grandes son las más frecuentes en los titulares analizados. Si `trabajo` domina con gran diferencia, el corpus incluye muchas noticias genéricas sobre empleo. Si `paro` o `huelga` son las más grandes, el período estuvo marcado por conflictos activos.

**Justificación.** Las nubes de palabras tienen limitaciones metodológicas conocidas: no muestran contexto, no distinguen connotaciones y pueden privilegiar términos morfológicamente simples. Se usan aquí con el propósito específico de comunicación accesible, complementando los análisis de frecuencia más rigurosos de las celdas anteriores (Heimerl et al., 2014).

---

### Celda 20 — Resumen ejecutivo

**Descripción.** Compila nueve métricas clave del corpus en una tabla compacta exportada como `.xlsx`: total de noticias antes y después de la deduplicación, cantidad de diarios monitoreados, categorías detectadas, palabras clave únicas, promedio de KW por noticia, densidad máxima registrada, categoría dominante y diario más activo.

**Uso.** Sirve como síntesis del análisis para compartir con colaboradores o incluir en reportes. El archivo puede abrirse en Excel o Google Sheets sin necesidad de ejecutar nuevamente el notebook.

**Interpretación.** Las métricas se leen de forma comparativa entre períodos o ejecuciones del proyecto. La diferencia entre "total noticias" y "noticias únicas" indica el nivel de duplicación entre diarios: si un mismo evento es cubierto por varios medios, aparecerá múltiples veces con titulares ligeramente distintos.

**Justificación.** Un resumen ejecutivo es estándar metodológico en investigación aplicada. Permite que otros investigadores o actores institucionales evalúen la representatividad del corpus sin necesidad de revisar el notebook completo, siguiendo los principios de transparencia y reproducibilidad (Stodden et al., 2014).

---

## Fase V — Análisis avanzados

### Celda 21 — Perfil editorial por diario

**Descripción.** Calcula la distribución porcentual de categorías temáticas dentro de cada diario usando `pd.crosstab` con `normalize="index"`, dividiendo cada valor por el total de noticias de ese diario. El resultado se visualiza como un gráfico de barras apiladas al 100% y se complementa con una tabla de la categoría dominante por medio.

**Uso.** Es el análisis central para comparar la agenda editorial de cada diario. A diferencia de la frecuencia absoluta, la normalización por fila permite comparar medios con distinto número de noticias en un pie de igualdad.

**Interpretación.** Un diario cuya barra está dominada por un solo color tiene una agenda laboral especializada hacia ese tipo de conflicto. Un diario con colores distribuidos parejamente tiene una cobertura más diversa. Las diferencias entre medios pueden reflejar distintas concepciones editoriales de qué es noticiable en el mundo laboral.

**Justificación.** El concepto de perfil editorial es central en la sociología de los medios y el periodismo comparado. La normalización por diario es metodológicamente necesaria para eliminar el efecto tamaño: sin ella, un diario con 200 noticias siempre parecería cuantitativamente más relevante que uno con 50, aunque su distribución temática sea idéntica (Breed, 1955).

---

### Celda 22 — Red de co-ocurrencia

**Descripción.** Construye un grafo no dirigido con NetworkX donde cada nodo es una palabra clave y cada arista conecta dos términos que aparecieron juntos en el mismo titular al menos N veces (umbral configurable). El grosor de las aristas refleja la frecuencia del par. Calcula la centralidad de grado de cada nodo para identificar los términos más articuladores de la red.

**Uso.** Permite ir más allá del análisis de frecuencias individuales para descubrir la estructura semántica del corpus: qué términos actúan como conectores que articulan múltiples dimensiones del discurso laboral.

**Interpretación.** Un nodo con alta centralidad de grado es un término que co-ocurre con vocabulario muy variado: es conceptualmente central en el discurso. Clústeres de nodos densamente conectados entre sí forman campos semánticos, grupos de palabras que los medios usan en los mismos contextos. Nodos aislados son términos que los medios usan en contextos muy específicos.

**Justificación.** El análisis de redes semánticas es una metodología consolidada en lingüística computacional y humanidades digitales. Aplicada a corpus periodísticos, permite identificar la estructura implícita del discurso: qué asociaciones de ideas se dan por sentadas y qué conceptos se presentan como relacionados (Manning & Schütze, 1999). El umbral de co-ocurrencia debe calibrarse según el tamaño del corpus: corpora pequeños requieren umbrales más bajos para no obtener una red vacía.

---

### Celda 23 — Índice de diversidad temática (entropía de Shannon)

**Descripción.** Calcula la entropía de Shannon para la distribución de categorías temáticas de cada diario. La fórmula es `H = -∑ pᵢ · log₂(pᵢ)` donde `pᵢ` es la proporción de noticias de cada categoría dentro del diario. El valor máximo teórico es `log₂(n_categorías)` y corresponde a una distribución perfectamente uniforme.

**Uso.** Cuantifica en una única cifra comparable cuán variada es la cobertura laboral de cada medio. Complementa el perfil editorial (celda 21) con un indicador sintético y objetivable.

**Interpretación.** Un diario con entropía cercana al máximo teórico cubre todos los tipos de conflicto con frecuencias similares: tiene una agenda diversa. Un diario con entropía baja concentra casi toda su cobertura en una o dos categorías. La diferencia entre el diario más y el menos diverso puede interpretarse como la *brecha de agenda* entre medios.

**Justificación.** La entropía de Shannon es el indicador estándar de diversidad en teoría de la información (Shannon, 1948), ampliamente adoptado en ecología, economía y ciencias de la comunicación para medir concentración y pluralismo. Su ventaja sobre medidas más simples (como el índice de Herfindahl) es que es sensible tanto a la distribución de frecuencias como al número de categorías posibles.

---

### Celda 24 — Actores sindicales mencionados

**Descripción.** Cruza el corpus contra un diccionario de organizaciones sindicales y laborales (CGT, ATE, UTA, UEPC, SUOEM, ADIUC, plataformas de economía de plataforma, etc.) para contar en cuántas noticias aparece cada actor y en cuántos diarios es nombrado. La visualización muestra barras ordenadas por menciones con anotaciones del número de medios que cubren cada organización.

**Uso.** Identifica qué organizaciones tienen mayor visibilidad mediática, cuáles son ignoradas por la prensa, y si existe un patrón de concentración: ciertos actores solo cubiertos por ciertos medios. El CSV exportado permite profundizar el análisis cualitativo por actor.

**Interpretación.** Una organización con muchas menciones pero presencia en pocos diarios tiene visibilidad concentrada. Una organización con pocas menciones pero en muchos diarios tiene presencia dispersa pero amplia. Ambos patrones tienen implicaciones distintas para la construcción de agenda pública.

**Justificación.** La visibilidad de los actores sociales en los medios es un indicador de poder simbólico: las organizaciones que no son nombradas tienen menor capacidad de instalar sus demandas en la agenda pública (Bourdieu, 1991). Medir esa visibilidad cuantitativamente permite objetivar lo que de otro modo sería una impresión subjetiva del investigador.

---

### Celda 25 — Score de relevancia y ranking de noticias

**Descripción.** Construye un índice compuesto de relevancia ponderando cuatro dimensiones normalizadas al rango [0, 1] mediante normalización min-max:

| Componente | Peso | Lógica |
|---|---|---|
| Densidad de palabras clave | 40% | Más términos laborales = más central al tema |
| Actores sindicales mencionados | 25% | Más actores = más institucional |
| Gravedad de la categoría | 25% | Huelga > corte > salario > desempleo |
| Longitud del titular | 10% | Más descriptivo = más informativo |

**Uso.** Permite seleccionar automáticamente las noticias más significativas del corpus para análisis cualitativo manual. En lugar de revisar cientos de titulares, el investigador puede concentrarse en el top 20 o top 50.

**Interpretación.** Un score alto indica una noticia que combina múltiples palabras clave laborales, menciona al menos un actor sindical, pertenece a una categoría de alta conflictividad y tiene un titular extenso e informativo. La distribución del score por diario revela si algún medio publica sistemáticamente noticias más densas y relevantes.

**Justificación.** Los índices compuestos son una herramienta estándar en ciencias sociales cuando no existe una sola variable que capture la relevancia de un fenómeno multidimensional. Los pesos asignados son decisiones metodológicas que deben explicitarse y pueden ajustarse según la hipótesis de investigación. La normalización min-max garantiza que ninguna variable domine por su escala de medición (Greco et al., 2019).

---

## Fase VI — Análisis temporal e histórico

### Celda 27 — Noticias vs. tiempo (análisis temporal completo)

**Descripción.** Genera tres niveles de granularidad temporal en un mismo conjunto de gráficos: serie diaria con media móvil de 7 días (panel A), histograma semanal (panel B) e histograma mensual (panel C). Complementariamente produce dos gráficos adicionales: evolución diaria desagregada por diario en líneas, y composición temática mensual en área apilada.

**Uso.** Es el análisis temporal principal del corpus histórico. Las tres granularidades responden a preguntas distintas: el gráfico diario detecta eventos puntuales, el semanal muestra ritmos de cobertura eliminando el ruido de fin de semana, y el mensual revela tendencias estructurales de mediano plazo.

**Interpretación.** La combinación de los tres paneles permite distinguir entre tipos de variación: un pico en el gráfico diario que no se ve en el semanal fue un evento puntual sin sostenimiento. Un alza durante varias semanas visible en el mensual indica un conflicto prolongado o una tendencia editorial. El área apilada mensual permite ver si la composición temática cambia con el tiempo.

**Justificación.** El análisis multiescalar del tiempo es una práctica estándar en periodismo de datos y ciencias sociales computacionales. Ninguna granularidad sola es suficiente: el dato diario tiene demasiado ruido, el mensual puede ocultar eventos significativos (Hyndman & Athanasopoulos, 2021).

---

### Celda 28 — Comparación entre períodos

**Descripción.** Divide el corpus en dos períodos en torno a una `FECHA_CORTE` configurable y compara métricas clave entre ambos: total de noticias, días cubiertos, diarios activos, categorías únicas, media de KW por noticia, mediana de longitud de titular e intensidad diaria (noticias/día). El gráfico presenta tres paneles comparativos: totales, intensidad y distribución de categorías.

**Uso.** Diseñado para evaluar si un evento exógeno (cambio de gobierno, decreto económico, paro nacional, reforma laboral) modificó el volumen o la composición de la cobertura laboral. La `FECHA_CORTE` puede ajustarse para cualquier hito relevante para la hipótesis del investigador.

**Interpretación.** Un aumento significativo de "noticias por día" en el período posterior indica que el evento de referencia intensificó la cobertura laboral. Un cambio en la distribución de categorías puede indicar un giro en el tipo de conflicto dominante. La métrica de "media de KW por noticia" es especialmente sensible: si aumenta tras el corte, los titulares posteriores son más ricos en vocabulario laboral.

**Justificación.** El diseño pre-post es el más simple de los diseños cuasi-experimentales en ciencias sociales. Aunque no permite establecer causalidad, sí permite documentar covariación temporal entre un evento y la cobertura mediática, que es la pregunta central de estudios de agenda-setting en contextos de cambio político o económico (McCombs & Shaw, 1972).

---

### Celda 29 — Detección de picos de conflictividad

**Descripción.** Identifica automáticamente los días con cobertura anormalmente alta usando un criterio estadístico: un día es "pico" cuando su cantidad de noticias supera `media + umbral × σ`. El umbral (por defecto 1.5σ) es configurable. Para los 5 picos más grandes, imprime las noticias que los componen con diario y categoría. Exporta un archivo `.xlsx` con todas las noticias de los días pico.

**Uso.** Permite hacer análisis retrospectivo de coyuntura: dado que hubo un pico de cobertura en una fecha, ¿qué eventos lo explican? El archivo exportado es el insumo para revisión cualitativa manual focalizada, concentrando la atención en los momentos de mayor actividad mediática.

**Interpretación.** Un pico con 80% de noticias de `huelga_paro` en un único diario puede ser un evento sectorial cubierto en profundidad por ese medio. Un pico con distribución equilibrada entre diarios y categorías indica un evento de impacto amplio como un paro general. El umbral de 1.5σ detecta aproximadamente el 7% de los días más activos; subir a 2σ selecciona solo el 2.5% más extremo.

**Justificación.** La detección de anomalías estadísticas es una técnica estándar en análisis de series temporales. En el análisis de cobertura mediática, los picos son equivalentes a lo que los sociólogos llaman *oleadas de protesta*: momentos de intensificación de la acción colectiva o de su representación mediática. Identificarlos automáticamente permite priorizar el trabajo cualitativo sin depender del conocimiento previo del investigador sobre el período (Tarrow, 2011).

---

### Celda 30 — Evolución de actores sindicales en el tiempo

**Descripción.** Para cada actor del diccionario `ACTORES`, crea una columna binaria que indica si ese actor fue mencionado en cada titular. Agrupa por mes y suma las presencias, construyendo una matriz actor × mes. Produce dos visualizaciones: líneas de evolución mensual y un heatmap actor × mes que actúa como calendario de visibilidad.

**Uso.** Permite rastrear la trayectoria mediática de organizaciones específicas a lo largo del tiempo. Es especialmente útil para analizar si la irrupción de un sindicato en los medios coincide con un conflicto particular, con períodos de paritarias o con eventos políticos externos.

**Interpretación.** Una línea de evolución con picos discretos indica que el actor solo aparece en coyunturas específicas. Una línea alta y estable indica que ese actor es una referencia permanente en la cobertura laboral. El heatmap permite identificar qué meses concentran la presencia de cada organización, facilitando la correlación con el calendario de conflictividad real.

**Justificación.** El concepto de *trayectoria mediática* de un actor social es central en el análisis longitudinal de movimientos sociales y organizaciones gremiales. La visibilidad de un sindicato en los medios no es estable: fluctúa según su actividad, la coyuntura política y las prioridades editoriales. Medir esa fluctuación permite distinguir actores con capacidad de instalar agenda de forma sostenida de aquellos que solo emergen en momentos de conflicto agudo (Bourdieu, 1991; Tarrow, 2011).

---

### Celda 31 — Sectores afectados vs. sectores movilizados

**Descripción.** Clasifica cada titular según su encuadre predominante en cuatro categorías mutuamente excluyentes: `movilizado` (el sector laboral actúa, reclama, se organiza), `afectado` (el sector padece algo externo como despidos, ajuste o crisis), `ambos` (coexisten los dos encuadres) y `sin_encuadre`. La clasificación se realiza con dos expresiones regulares independientes que capturan vocabulario de acción versus vocabulario de padecimiento. Produce tres paneles: torta general, barras apiladas por diario y evolución mensual del encuadre. Exporta el dataset completo con la columna `encuadre` para análisis cualitativo posterior.

**Uso.** Es el análisis cualitativo más sofisticado del proyecto. Permite estudiar el *framing* de la conflictividad laboral: ¿los diarios presentan a los trabajadores como víctimas pasivas de decisiones económicas o como actores que se organizan y luchan? Esta distinción es central en el análisis crítico del discurso y en los estudios de representación mediática del trabajo.

**Interpretación.** Si la categoría `afectado` domina el corpus, los medios encuadran la conflictividad laboral predominantemente desde la perspectiva de la vulnerabilidad. Si `movilizado` es más frecuente, los medios dan visibilidad a la agencia colectiva de los trabajadores. La evolución mensual permite ver si el encuadre cambia con la coyuntura. Las diferencias entre diarios revelan distintas concepciones editoriales de qué es el conflicto laboral.

**Justificación.** La teoría del framing (Entman, 1993; Gitlin, 1980) sostiene que los medios no solo informan sobre la realidad sino que la enmarcan en narrativas que definen quiénes son los protagonistas, cuáles son las causas y qué soluciones son legítimas. En el caso del conflicto laboral, el encuadre `afectado` naturaliza la posición de víctima y diluye la responsabilidad de los actores económicos; el encuadre `movilizado` reconoce la agencia colectiva de los trabajadores. Operacionalizar esta distinción mediante expresiones regulares es una forma de escalar el análisis de framing a corpora de cientos o miles de titulares, tarea que de otro modo requeriría codificación manual con protocolos de acuerdo inter-juez.

---

## Referencias metodológicas

- Bourdieu, P. (1991). *Language and Symbolic Power*. Harvard University Press.
- Breed, W. (1955). Social control in the newsroom: A functional analysis. *Social Forces*, 33(4), 326–335.
- Cairo, A. (2016). *The Truthful Art: Data, Charts, and Maps for Communication*. New Riders.
- Entman, R. M. (1993). Framing: Toward clarification of a fractured paradigm. *Journal of Communication*, 43(4), 51–58.
- Fairclough, N. (1989). *Language and Power*. Longman.
- Firth, J. R. (1957). A synopsis of linguistic theory 1930–1955. *Studies in Linguistic Analysis*, 1–32.
- Gitlin, T. (1980). *The Whole World Is Watching*. University of California Press.
- Greco, S., Ishizaka, A., Tasiou, M., & Torrisi, G. (2019). On the methodological framework of composite indices. *Social Indicators Research*, 141(2), 637–676.
- Heimerl, F., Lohmann, S., Lange, S., & Ertl, T. (2014). Word cloud explorer. In *47th Hawaii International Conference on System Sciences* (pp. 1833–1842). IEEE.
- Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and Practice* (3rd ed.). OTexts. [https://otexts.com/fpp3](https://otexts.com/fpp3)
- Knuth, D. E. (1984). Literate programming. *The Computer Journal*, 27(2), 97–111.
- Krippendorff, K. (2018). *Content Analysis: An Introduction to Its Methodology* (4th ed.). SAGE.
- Manning, C. D., & Schütze, H. (1999). *Foundations of Statistical Natural Language Processing*. MIT Press.
- McCombs, M., & Shaw, D. (1972). The agenda-setting function of mass media. *Public Opinion Quarterly*, 36(2), 176–187.
- Neuendorf, K. A. (2017). *The Content Analysis Guidebook* (2nd ed.). SAGE.
- Shannon, C. E. (1948). A mathematical theory of communication. *Bell System Technical Journal*, 27(3), 379–423.
- Stodden, V., Leisch, F., & Peng, R. D. (2014). *Implementing Reproducible Research*. CRC Press.
- Tarrow, S. (2011). *Power in Movement: Social Movements and Contentious Politics* (3rd ed.). Cambridge University Press.
- Tufte, E. R. (1983). *The Visual Display of Quantitative Information*. Graphics Press.
- Tukey, J. W. (1977). *Exploratory Data Analysis*. Addison-Wesley.
