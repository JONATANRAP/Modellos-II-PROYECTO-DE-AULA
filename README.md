# Sistema Predictivo de Recomendación de Cultivos para la Optimización Agrícola

## 1. Descripción del Problema

La selección adecuada de un cultivo depende de la interacción multivariable y no lineal entre la disponibilidad de nutrientes del suelo y las condiciones climáticas del terreno. Este proyecto implementa un sistema predictivo de clasificación multiclase supervisada que recomienda el cultivo más apto entre **22 clases balanceadas** (100 registros por cultivo, $N = 2200$ muestras sin valores faltantes ni duplicados) a partir de **7 variables edafoclimáticas numéricas**:

| Variable | Descripción | Unidad |
| :--- | :--- | :--- |
| **N** | Nitrógeno disponible en el suelo | kg/ha\* |
| **P** | Fósforo disponible en el suelo | kg/ha\* |
| **K** | Potasio disponible en el suelo | kg/ha\* |
| **temperature** | Temperatura ambiente | °C |
| **humidity** | Humedad relativa del aire | % |
| **ph** | Acidez o alcalinidad del suelo | adimensional (escala 0–14) |
| **rainfall** | Precipitación | mm |

\* La fuente describe N, P y K como la proporción (*ratio*) del contenido de cada nutriente en el suelo, sin indicar explícitamente la unidad; en la literatura que utiliza este conjunto de datos se interpretan como kg/ha.

El flujo experimental abarca desde el análisis exploratorio estadístico hasta el ajuste de hiperparámetros de cinco familias de clasificadores sin fuga de información (*data leakage*). **La selección de modelos, de variables y de la representación (original, PCA o UMAP) se realiza exclusivamente con datos de validación**: con la mejor configuración de cada modelo se repite la partición en $K = 5$ pliegues y se calcula el intervalo de confianza de Wilson al 95 % de la exactitud de validación. El conjunto de prueba se usa una sola vez, al final, para evaluar el modelo elegido.

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
│   ├── referencias.bib
│   └── figuras/   (generadas por los notebooks 02 y 03)
├── requirements.txt
└── README.md
```

- **`data/`**: Almacena el conjunto de datos original `Crop_recommendation.csv` con las 2200 observaciones y las 22 etiquetas de cultivo.
- **`notebooks/`**: Contiene los tres cuadernos Jupyter estructurados secuencialmente con explicaciones metodológicas, código reproducible (`random_state=42`) y resultados ejecutados.
- **`informe/`**: Contiene el código fuente en LaTeX bajo formato IEEE de doble columna, la base bibliográfica BibTeX (`referencias.bib`), las figuras en PDF (`figuras/`, generadas al ejecutar los notebooks 02 y 03) y el documento compilado en PDF.
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

2. **`notebooks/02_Modelado_y_Evaluacion_Base.ipynb` (Modelado y selección de modelos con validación):**
   - Partición estratificada 80/20 (1760 muestras de entrenamiento y 440 de prueba, `random_state=42`). El conjunto de prueba **no se usa** en este notebook.
   - Encapsulamiento de `StandardScaler` dentro de `Pipeline` para evitar fuga de datos.
   - Ajuste de hiperparámetros mediante `GridSearchCV` con `StratifiedKFold(n_splits=5)` para cinco modelos: Regresión Logística, KNN, Random Forest, Perceptrón Multicapa (MLP) y SVM con núcleo RBF.
   - Validación de la mejor configuración de cada modelo repitiendo la partición en $K = 5$ pliegues (`random_state=7`) e intervalo de confianza de Wilson al 95 % ($n = 1760$). Selección de los dos mejores modelos (Random Forest y SVM RBF) y matriz de confusión de validación.

3. **`notebooks/03_Reduccion_Dimensionalidad_PCA_UMAP.ipynb` (Selección de características, reducción de dimensionalidad y evaluación final):**
   - Análisis individual de variables: importancia de Gini, exactitud de validación usando una sola variable y matrices de confusión por variable.
   - Selección secuencial hacia adelante (SFS) con validación $K = 5$.
   - PCA ($k = 6$ componentes, 97.24 % de la varianza) y UMAP (2 componentes, no supervisado) dentro del `Pipeline` de validación.
   - Comparación integral en validación y **evaluación única en prueba** del modelo elegido, con IC de Wilson y matriz de confusión.

---

## 5. Resultados Principales

### Selección de modelos (validación, $K = 5$, $n = 1760$)

| Enfoque / Modelo | Representación | Exactitud validación | F1 macro | Errores | IC Wilson 95 % |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Base: Random Forest** (`n_estimators=19`, `max_depth=10`, `sqrt`) | **Original (7 variables)** | **99.49 % ± 0.37** | **0.9949** | **9** | **[99.03 %, 99.73 %]** |
| SFS: Random Forest | 5 variables (rainfall, humidity, K, N, P) | 99.03 % ± 0.25 | 0.9903 | 17 | [98.46 %, 99.40 %] |
| Base: SVM RBF (`C=1`, `gamma=1`) | Original (7 variables) | 98.64 % ± 0.58 | 0.9864 | 24 | [97.98 %, 99.08 %] |
| Base: MLP (`hidden_layer_sizes=(32,)`, `alpha=0.01`) | Original (7 variables) | 98.41 % ± 0.95 | 0.9841 | 28 | [97.71 %, 98.90 %] |
| SFS: SVM RBF | 5 variables (rainfall, humidity, K, N, P) | 98.12 % ± 0.48 | 0.9812 | 33 | [97.38 %, 98.66 %] |
| PCA + SVM RBF | 6 componentes (97.24 % var.) | 98.07 % ± 0.55 | 0.9807 | 34 | [97.31 %, 98.61 %] |
| Base: Regresión Logística (`C=100`) | Original (7 variables) | 98.07 % ± 0.68 | 0.9805 | 34 | [97.31 %, 98.61 %] |
| Base: KNN (`n_neighbors=4`) | Original (7 variables) | 96.82 % ± 0.88 | 0.9679 | 56 | [95.89 %, 97.54 %] |
| UMAP + SVM RBF | 2 componentes no lineales | 94.55 % ± 1.16 | 0.9444 | 96 | [93.38 %, 95.51 %] |
| PCA + Random Forest | 6 componentes (97.24 % var.) | 94.32 % ± 0.57 | 0.9433 | 100 | [93.14 %, 95.31 %] |
| UMAP + Random Forest | 2 componentes no lineales | 93.92 % ± 1.52 | 0.9386 | 107 | [92.71 %, 94.94 %] |

### Evaluación final (prueba, una sola vez, $n = 440$)

| Modelo final | Exactitud | Error | Precisión macro | Sensibilidad macro | F1 macro | IC Wilson 95 % |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| Random Forest (7 variables) | 98.86 % (435/440) | 1.14 % | 0.9897 | 0.9886 | 0.9887 | [97.37 %, 99.51 %] |

### Síntesis de Hallazgos

- **Mejor modelo según validación:** Random Forest con las 7 variables originales (99.49 %). Su ventaja sobre MLP, Regresión Logística y KNN es estadísticamente significativa (intervalos de Wilson sin solapamiento); frente a SVM RBF los intervalos prácticamente se tocan.
- **Análisis individual de variables:** ninguna variable es suficiente por sí sola; la mejor (`rainfall`) alcanza 36.6 % (RF) y 37.9 % (SVM). Las matrices de confusión muestran que cada variable reconoce pocos cultivos (por ejemplo, `K` identifica garbanzo y naranja, pero confunde manzana y uva, que tienen niveles idénticos de potasio).
- **Selección de características:** con `rainfall`, `humidity`, `K`, `N` y `P` se obtiene un desempeño estadísticamente equivalente al modelo completo; `temperature` y `ph` podrían omitirse.
- **PCA y UMAP:** no mejoran el desempeño. PCA conserva el de SVM (98.07 %) pero degrada el de Random Forest (94.32 %); UMAP es útil para visualización (≈ 94 %).
- **Prueba:** el modelo final obtiene 98.86 % en prueba, coherente con la estimación de validación.

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
