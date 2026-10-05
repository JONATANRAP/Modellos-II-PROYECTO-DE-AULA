# ml-crop-recommendation

Sistema predictivo basado en aprendizaje automático para la recomendación de cultivos a partir de propiedades fisicoquímicas del suelo y variables meteorológicas.

##  Descripción del Proyecto

El proyecto aborda la recomendación de cultivos formulada como una tarea de clasificación multiclase supervisada. A partir de siete variables agroclimáticas (nitrógeno, fósforo, potasio, temperatura, humedad relativa, pH y precipitación), se evalúan diferentes familias de clasificadores con el propósito de sugerir el cultivo óptimo entre 22 alternativas balanceadas.

## 👥 Autores

* **Jonatan Romero** - Ingeniería de Sistemas, Universidad de Antioquia (Medellín, Antioquia) — [jonatan.romeroa@udea.edu.co](mailto:jonatan.romeroa@udea.edu.co)
* **Sebastian Berrio** - Ingeniería de Sistemas, Universidad de Antioquia (Medellín, Antioquia) — [sebastian.berriom@udea.edu.co](mailto:sebastian.berriom@udea.edu.co)
* **Cristian Diez** - Ingeniería de Sistemas, Universidad de Antioquia (Medellín, Antioquia) — [cristian.diez@udea.edu.co](mailto:cristian.diez@udea.edu.co)

## 📁 Estructura del Repositorio

- `data/`: Contiene el conjunto de datos de referencia (`Crop_recommendation.csv`).
- `notebooks/`: Cuaderno Jupyter/Colab con el análisis exploratorio de datos (EDA) y la preparación del pipeline.
- `informe/`: Documento final del proyecto en formato IEEE (`documento_entrega.pdf`).
- `requirements.txt`: Lista de dependencias necesarias para ejecutar el proyecto.

## ⚙️ Requisitos e Instalación

El código fue desarrollado en **Python 3.10+**. Para instalar las dependencias necesarias:

```bash
git clone https://github.com/crd10lop/ml-crop-recommendation.git
cd ml-crop-recommendation
pip install -r requirements.txt
```
