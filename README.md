# Evaluación de la Relevancia Informativa de Características en Datasets CAPEC

> **Trabajo Fin de Máster (TFM)** — *Evaluación de la Relevancia Informativa de Características en Datasets basados en CAPEC de Ataques Cibernéticos Mediante Entropía de Columnas*

Herramienta de análisis en Python para medir la **relevancia informativa** de las columnas en datasets de patrones de ataque cibernético, aplicando **entropía de Shannon** y la métrica compuesta **EIAC** (*Entropía Informativa Ajustada por Columna*).

---

## Descripción

Este repositorio forma parte de un TFM orientado a evaluar qué atributos de un dataset aportan información útil para el análisis de patrones de ataque descritos en [CAPEC](https://capec.mitre.org/) (*Common Attack Pattern Enumeration and Classification*, MITRE).

El script `procesar_capec_entropia.py` automatiza el flujo completo:

1. **Ingesta** de un dataset en formato CSV.
2. **Enriquecimiento** con variables derivadas (conteos de elementos, longitudes de texto, etc.).
3. **Cálculo de entropía** por columna y ranking de variabilidad.
4. **Evaluación EIAC** con clasificación funcional y de relevancia.
5. **Generación de artefactos** listos para el TFM: CSV, tablas LaTeX y reportes PDF.

---

## Metodología


### Métrica EIAC

La entropía pura puede sobrevalorar columnas con muchos valores únicos o datos faltantes. **EIAC** ajusta la medida incorporando:

| Símbolo | Significado |
|---------|-------------|
| \(H_n(X)\) | Entropía normalizada |
| \(C(X)\) | Completitud (proporción de valores no vacíos) |
| \(U(X)\) | Ratio de unicidad |
| **EIAC** | \(H_n(X) * C(X) * (1 - U(X))\) |

### Clasificación de relevancia

| Rango EIAC | Clasificación |
|------------|---------------|
| ≥ 0.66 | Alta relevancia |
| 0.33 – 0.66 | Relevancia media |
| < 0.33 | Baja relevancia |

Además, cada columna recibe una **clasificación funcional** (analítica, trazabilidad, descriptiva contextual, bajo aporte por datos faltantes, etc.) y un texto de análisis interpretativo.

---

## Estructura del proyecto

```
capec-v3.9/
├── procesar_capec_entropia.py   # Script principal de análisis
├── capecnuevo.xml               # Catálogo CAPEC v3.9 (MITRE)
├── datasets/                    # Datasets de entrada
│   ├── CAPEC/                   # Dataset basado en el catálogo CAPEC
│   ├── sintetico/               # Dataset sintético generado
│   └── SR-BH/                   # Dataset SR-BH (referencia comparativa)
└── datos/                       # Salidas generadas por el script
    ├── processed/               # CSV enriquecido
    └── results/                 # Rankings, LaTeX y PDF
```

Cada subcarpeta de `datasets/` debe contener **un archivo CSV** con el dataset correspondiente. El análisis se ejecuta indicando la ruta del CSV con el parámetro `--input`.

---

## Requisitos

- **Python** 3.9 o superior
- **pandas** — manipulación de datos
- **reportlab** — generación de reportes PDF *(opcional pero recomendado)*

```bash
python3 -m pip install pandas reportlab
```

---

## Uso

### Ejecución básica (fuente remota por defecto)

```bash
python3 procesar_capec_entropia.py
```

Por defecto, el script descarga el dataset desde una hoja de Google Sheets publicada como CSV.

### Análisis sobre datasets locales

```bash
# Dataset CAPEC
python3 procesar_capec_entropia.py --input datasets/CAPEC/dataset.csv

# Dataset sintético
python3 procesar_capec_entropia.py --input datasets/sintetico/dataset.csv

# Dataset SR-BH
python3 procesar_capec_entropia.py --input datasets/SR-BH/dataset.csv
```

### Parámetros principales

| Parámetro | Descripción | Valor por defecto |
|-----------|-------------|-------------------|
| `--input` | Ruta local o URL del CSV de entrada | Google Sheets (CAPEC) |
| `--processed` | Ruta del dataset procesado | `datos/processed/capec_dataset.csv` |
| `--entropy` | Ranking de entropía | `datos/results/entropia_columnas.csv` |
| `--eiac` | Ranking EIAC | `datos/results/eiac_columnas.csv` |
| `--eiac-latex` | Tabla EIAC en LaTeX | `datos/results/eiac_tabla_latex.tex` |
| `--eiac-pdf` | Reporte EIAC en PDF | `datos/results/eiac_reporte.pdf` |
| `--comparison-pdf` | Comparación Hₙ(X) vs EIAC (PDF) | `datos/results/eiac_comparacion_hn_eiac.pdf` |
| `--comparison-latex` | Comparación Hₙ(X) vs EIAC (LaTeX) | `datos/results/eiac_comparacion_hn_eiac.tex` |
| `--pdf-rows` | Filas máximas en el PDF | Todas |
| `--print-rows` | Filas a mostrar en consola | Todas |

Cada ejecución añade un **timestamp** a los archivos de salida (formato `YYYYMMDD_HHMMSS`) para conservar el historial de análisis.

---

## Salidas generadas

| Archivo | Contenido |
|---------|-----------|
| `capec_dataset_*.csv` | Dataset original + columnas derivadas (conteos, longitudes) |
| `entropia_columnas_*.csv` | Entropía de Shannon, entropía normalizada y clasificación por variabilidad |
| `eiac_columnas_*.csv` | Ranking EIAC completo con distribución, relevancia y clasificación funcional |
| `eiac_tabla_latex_*.tex` | Tabla LaTeX lista para incluir en la memoria del TFM |
| `eiac_reporte_*.pdf` | Reporte visual con campos obligatorios y columnas complementarias |
| `eiac_comparacion_hn_eiac_*.pdf` | Comparación entre entropía normalizada y EIAC |
| `eiac_comparacion_hn_eiac_*.tex` | Tabla LaTeX de la comparación Hₙ(X) vs EIAC |

### Columnas de análisis obligatorio

El reporte PDF destaca un subconjunto de columnas definido como núcleo del dataset modelo:

- `ID`, `Name`, `Description`
- `Abstraction`, `Status`
- `Likelihood Of Attack`, `Typical Severity`

---

## Procesamiento de datos

El script enriquece el dataset original extrayendo métricas de campos estructurados de CAPEC:

- **Longitudes** de `Description` y `Notes`
- **Conteos** de términos alternativos, prerrequisitos, mitigaciones, indicadores, etc.
- **Patrones** en `Execution Flow` (pasos y técnicas), `Consequences` (alcance e impacto) y `Taxonomy Mappings`

Estas columnas derivadas se incluyen en el análisis de entropía y EIAC junto con las columnas originales.

---

## Contexto académico

Este código implementa la parte experimental del TFM, permitiendo:

- **Comparar** la relevancia informativa entre datasets reales (CAPEC), sintéticos y de referencia (SR-BH).
- **Identificar** qué columnas aportan valor analítico frente a las que actúan como identificadores, descriptores contextuales o presentan baja variabilidad.
- **Contrastar** la entropía normalizada con EIAC para evidenciar el efecto de la completitud y la unicidad en la evaluación de características.
- **Exportar** resultados en formatos académicos (CSV, LaTeX, PDF) para su integración directa en la memoria.

---

## Referencias

- [MITRE CAPEC](https://capec.mitre.org/) — Common Attack Pattern Enumeration and Classification
- CAPEC Version 3.9 (2023-01-24) — incluido en `capecnuevo.xml`
- Shannon, C. E. (1948). *A Mathematical Theory of Communication*

---

## Licencia y uso

Proyecto académico desarrollado en el marco del **Máster en Big Data**. El catálogo CAPEC es propiedad de [The MITRE Corporation](https://www.mitre.org/) y se distribuye bajo sus condiciones de uso.
