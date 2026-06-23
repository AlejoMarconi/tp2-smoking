# TP2 - Predicción de fumadores

El objetivo es predecir si una persona es fumadora o no (`smoking` = 0/1) a partir de variables clínicas y demográficas, y luego aplicar ese modelo sobre un dataset sin etiquetar para generar el archivo de entrega.

La métrica de evaluación es el **F1-Score sobre la clase 1**.

## Estructura

```
tp2-smoking/
├── data/
│   ├── raw/           # datasets originales (.xlsx)
│   ├── processed/     # datos procesados + predictions.csv/xlsx
│   └── external/      # sin datos externos utilizados
├── models/            # preprocessor y modelo final (joblib)
├── notebooks/
│   ├── 01_lectura_y_discovery.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_preprocesamiento.ipynb
│   ├── 04_entrenamiento_y_optimizacion.ipynb
│   ├── 05_validacion.ipynb
│   └── 06_prediccion.ipynb
└── requirements.txt
```

## Cómo reproducir

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Ejecutar las notebooks en orden (01 → 06). Al terminar queda `data/processed/predictions.xlsx` con la columna `smoking_prediction`.

## Experimentos y decisiones tomadas

**Variable `oral`:** En una primera corrida la dejé como feature. No movió ninguna métrica y generó una columna constante post-OneHot, así que la saqué del pipeline.

**Duplicados:** Probé eliminar las filas duplicadas ignorando el ID. El F1 bajó marginalmente y además no tiene sentido hacerlo: en producción no hay forma de saber si un registro nuevo "ya existía". Se mantienen todos los registros.

**Desbalance de clases:** El target tiene 63% no fumadores / 37% fumadores. Probé con SMOTE pero no mejoró respecto a simplemente usar `class_weight='balanced'` en los modelos que lo soportan. Opté por la opción más simple.

**Comparación de modelos:** Comparé cuatro baselines (Logistic Regression, Decision Tree, Random Forest, XGBoost). Random Forest y XGBoost quedaron muy cerca en F1. Elegí XGBoost porque converge más rápido al optimizar hiperparámetros con GridSearchCV.

**Optimización:** Hice GridSearchCV con CV=3 sobre los hiperparámetros principales de XGBoost (profundidad, learning rate, número de estimadores). Usé CV=3 en vez de 5 para que la búsqueda sea más rápida sin perder confiabilidad dado el tamaño del dataset.

**Outliers:** Hay valores extremos en variables como `triglyceride` y `Gtp`. Decidí no recortarlos porque son valores fisiológicamente posibles, XGBoost es robusto a outliers, y cualquier regla de recorte tendría que replicarse exactamente en predicción.

## Conclusiones

- El modelo final es XGBoost optimizado con GridSearchCV, encapsulado en un Pipeline junto con el preprocesador.
- F1-Score sobre clase 1 en test: **0.70**. ROC AUC: **0.86**.
- La variable más importante es `gender` (categórica), seguida por `hemoglobin`, `Gtp` y `triglyceride`. Esto no coincide con la correlación lineal con el target, que subestima a las variables categóricas.
- Sobre el dataset sin etiquetar se predijeron 2.207 fumadores de 5.692 personas (38.8%), consistente con la distribución del set de entrenamiento (37%).
- El archivo de entrega es `data/processed/predictions.xlsx` con la columna `smoking_prediction` en valores 0/1.
