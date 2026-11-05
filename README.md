# Sistema Predictivo de Recomendación de Cultivos para la Optimización Agrícola

## 1. Descripción del Problema

La selección adecuada de un cultivo depende de la interacción multivariable y no lineal entre la disponibilidad de nutrientes del suelo y las condiciones climáticas del terreno. Este proyecto implementa un sistema predictivo de clasificación multiclase supervisada que recomienda el cultivo más apto entre **22 clases balanceadas** (100 registros por cultivo, $N = 2200$ muestras sin valores faltantes ni duplicados) a partir de **7 variables edafoclimáticas numéricas**:

- **N:** Contenido de nitrógeno en el suelo.
- **P:** Contenido de fósforo en el suelo.
- **K:** Contenido de potasio en el suelo.
- **temperature:** Temperatura ambiente (°C).
- **humidity:** Humedad relativa (%).
- **ph:** Nivel de acidez o alcalinidad del suelo.
- **rainfall:** Precipitación acumulada (mm).

El flujo experimental abarca desde el análisis exploratorio estadístico hasta el ajuste de hiperparámetros de cinco familias de clasificadores mediante validación cruzada estratificada sin fuga de información (*data leakage*), complementado con la evaluación de importancia de características y técnicas de reducción de dimensionalidad lineal (PCA) y no lineal (UMAP).

---

## 2. Estructura del Repositorio

```text
.
├── data/
│   └── Crop_recommendation.csv
├── notebooks/
│   ├── 01_EDA_Recomendacion_Cultivos.ipynb
│   ├── 02_Modelado_y_Evaluacion_Base.ipynb
│   └── 03_Reduccion_Dimensionalidad_PCA_UMAP.ipynb
├── informe/
│   ├── INFORME_Sistema_Predictivo_de_Recomendación_de_Cultivos_para_la_Optimización_Agrícola.tex
│   ├── INFORME_Sistema_Predictivo_de_Recomendación_de_Cultivos_para_la_Optimización_Agrícola.pdf
│   └── referencias.bib
├── requirements.txt
└── README.md
```

- **`data/`**: Almacena el conjunto de datos original `Crop_recommendation.csv` con las 2200 observaciones y las 22 etiquetas de cultivo.
- **`notebooks/`**: Contiene los tres cuadernos Jupyter estructurados secuencialmente con explicaciones metodológicas, código reproducible (`random_state=42`) y resultados ejecutados.
- **`informe/`**: Contiene el código fuente en LaTeX bajo formato IEEE de doble columna, la base bibliográfica BibTeX (`referencias.bib`) y el documento compilado en PDF.
- **`requirements.txt`**: Especifica las librerías de Python necesarias para reproducir el entorno experimental.

---

## 3. Instalación y Configuración del Entorno

El proyecto requiere **Python 3.10 o superior**. Se recomienda crear un entorno virtual aislado antes de instalar las dependencias:

### En Windows (PowerShell)

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### En Linux / macOS

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

## 4. Guía de Ejecución Secuencial de los Notebooks

Los cuadernos están diseñados para ejecutarse desde la raíz del proyecto o dentro de VS Code / Jupyter Lab manteniendo la ruta relativa `data/Crop_recommendation.csv`:

1. **`notebooks/01_EDA_Recomendacion_Cultivos.ipynb` (Análisis Exploratorio de Datos):**
   - Verificación de integridad estructural, ausencia de valores nulos y balance exacto de clases ($\text{IR} = 1.00$).
   - Estadísticos descriptivos, asimetría, curtosis y prueba ANOVA de una vía ($F$) para estimar la separabilidad univariada.
   - Detección de valores atípicos mediante el criterio $1.5 \times \text{IQR}$, demostrando su carácter discriminante por cultivo, y análisis de colinealidad mediante correlación de Pearson ($r = 0.74$ entre `P` y `K`).

2. **`notebooks/02_Modelado_y_Evaluacion_Base.ipynb` (Modelado y Evaluación Base):**
   - Partición estratificada 80/20 (1760 muestras de entrenamiento y 440 de prueba, `random_state=42`).
   - Encapsulamiento de `StandardScaler` dentro de `Pipeline` para evitar fuga de datos durante la validación cruzada.
   - Optimización de hiperparámetros mediante `GridSearchCV` con `StratifiedKFold(n_splits=5)` para cinco modelos: Regresión Logística, KNN, Random Forest, Perceptrón Multicapa (MLP) y SVM con núcleo RBF.

3. **`notebooks/03_Reduccion_Dimensionalidad_PCA_UMAP.ipynb` (Reducción de Dimensionalidad y Comparación Integral):**
   - Extracción de importancia de características con Random Forest (reducción media de impureza de Gini), donde `rainfall` (20.41%), `humidity` (20.33%), `K` (19.57%) y `P` (17.02%) concentran el 77.33% del peso total.
   - Reducción lineal mediante PCA reteniendo $k = 6$ componentes principales que explican el 97.24% de la varianza acumulada.
   - Proyección no lineal mediante UMAP a 2 componentes (`n_neighbors=15`, `min_dist=0.1`, `n_components=2`) y reevaluación de los dos mejores clasificadores (MLP y SVM RBF).

---

## 5. Resultados Principales

La siguiente tabla consolida el desempeño sobre el conjunto de prueba (440 muestras) de los cinco modelos base en el espacio original y de los dos mejores clasificadores tras aplicar reducción de dimensionalidad con PCA y UMAP:

| Enfoque / Modelo | Dimensión | Config. Óptima / Varianza | Accuracy | Error (1 - Acc) | Macro Precision | Macro Recall | Macro F1 |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **Base: Perceptrón Multicapa (MLP)** | **Original (7 vars)** | **`hidden_layer_sizes=(32,)`, `alpha=0.01`** | **0.9932** | **0.0068** | **0.9937** | **0.9932** | **0.9932** |
| Base: SVM (RBF) | Original (7 vars) | `C=1`, `gamma=1` | 0.9909 | 0.0091 | 0.9915 | 0.9909 | 0.9909 |
| Base: Random Forest | Original (7 vars) | `n_estimators=19`, `max_depth=10`, `sqrt` | 0.9886 | 0.0114 | 0.9897 | 0.9886 | 0.9887 |
| Base: Regresión Logística | Original (7 vars) | `C=100` | 0.9841 | 0.0159 | 0.9852 | 0.9841 | 0.9840 |
| Base: K-Nearest Neighbors (KNN) | Original (7 vars) | `n_neighbors=4` | 0.9727 | 0.0273 | 0.9746 | 0.9727 | 0.9725 |
| **PCA + Perceptrón Multicapa (MLP)** | **Reducida (6 comp.)** | **97.24% varianza explicada** | **0.9886** | **0.0114** | **0.9889** | **0.9886** | **0.9886** |
| PCA + SVM (RBF) | Reducida (6 comp.) | 97.24% varianza explicada | 0.9841 | 0.0159 | 0.9847 | 0.9841 | 0.9840 |
| UMAP + SVM (RBF) | Reducida (2 comp.) | Proyección no lineal 2D | 0.9614 | 0.0386 | 0.9660 | 0.9614 | 0.9611 |
| UMAP + Perceptrón Multicapa (MLP) | Reducida (2 comp.) | Proyección no lineal 2D | 0.9318 | 0.0682 | 0.9406 | 0.9318 | 0.9294 |

### Síntesis de Hallazgos

- **Mejor modelo predictivo:** El Perceptrón Multicapa (MLP) sobre las 7 variables originales estandarizadas alcanzó el mayor rendimiento con **99.32% de exactitud** y **0.9932 de Macro F1** (solo 3 muestras mal clasificadas de 440), seguido de SVM RBF con **99.09% de exactitud**.
- **Efecto de PCA:** Al eliminar la séptima dirección ortogonal asociada a la colinealidad entre fósforo y potasio, las primeras 6 componentes retienen el 97.24% de la varianza y el MLP mantiene un **98.86% de exactitud** (variación de apenas 0.46 puntos porcentuales).
- **Efecto de UMAP:** La reducción no lineal a 2 dimensiones separa visualmente los 22 cultivos en el plano y conserva una exactitud del **96.14% en SVM RBF** y **93.18% en MLP**, resultando útil para visualización exploratoria, aunque con pérdida predictiva frente al espacio completo.

---

## 6. Informe Técnico y Autores

### Documentación del Informe (Formato IEEE)

- **Archivo fuente LaTeX:** [`informe/INFORME_Sistema_Predictivo_de_Recomendación_de_Cultivos_para_la_Optimización_Agrícola.tex`](informe/INFORME_Sistema_Predictivo_de_Recomendación_de_Cultivos_para_la_Optimización_Agrícola.tex)
- **Documento PDF compilado:** [`informe/INFORME_Sistema_Predictivo_de_Recomendación_de_Cultivos_para_la_Optimización_Agrícola.pdf`](informe/INFORME_Sistema_Predictivo_de_Recomendación_de_Cultivos_para_la_Optimización_Agrícola.pdf)
- **Referencias BibTeX:** [`informe/referencias.bib`](informe/referencias.bib)

### Autores

- **Jonatan Romero** — Ingeniería de Sistemas, Universidad de Antioquia (Medellín, Antioquia) — [jonatan.romeroa@udea.edu.co](mailto:jonatan.romeroa@udea.edu.co)
- **Sebastian Berrio** — Ingeniería de Sistemas, Universidad de Antioquia (Medellín, Antioquia) — [sebastian.berriom@udea.edu.co](mailto:sebastian.berriom@udea.edu.co)
- **Cristian Diez** — Ingeniería de Sistemas, Universidad de Antioquia (Medellín, Antioquia) — [cristian.diez@udea.edu.co](mailto:cristian.diez@udea.edu.co)
