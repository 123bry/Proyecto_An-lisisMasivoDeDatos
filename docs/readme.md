# Proyecto de Curso: Modelo de Siguiente Mejor Producto (Next Best Product)

**Asignatura:** Análisis de Datos Masivos  
**Semestre:** 10mo Semestre  
**Institución:** Universidad de Guayaquil  
**Estudiantes:** * Cadena Vallejo Daniel
* Casco Orozco Bryan
* Chiriguaya Soriano Kevin
* Herrera Diaz Brando
* Parrales Tubay Cristhian

---

## 1. Descripción General del Proyecto
Este proyecto tiene como objetivo desarrollar un flujo completo de datos y machine learning (pipeline) para construir un modelo de **Next Best Product** (Mejor Siguiente Producto a Ofrecer). El propósito del modelo es predecir qué producto financiero tiene mayor probabilidad de ser adquirido por un cliente en el siguiente periodo, a partir de su información histórica, productos actuales, segmento y comportamiento mensual.

## 2. Arquitectura de Datos Implementada
El flujo de datos se diseñó utilizando una arquitectura por capas (Medallion Architecture) para garantizar la trazabilidad y calidad de la información:

* **Capa Raw:** Contiene los archivos originales normalizados en formato CSV (`clientes.csv`, `provincias.csv`, `segmentos.csv`, `productos.csv`, `cliente_estado_mensual.csv`, `cliente_producto_mensual.csv`) tal como fueron recibidos, sirviendo como respaldo inmutable.
* **Capa Bronze:** Almacena los mismos datos de Raw pero convertidos al formato eficiente Parquet, incluyendo columnas técnicas de auditoría para mantener la trazabilidad del origen.
* **Capa Silver:** Contiene los datos limpios, tipados y relacionados. Aquí se aplicaron reglas de calidad, eliminación de duplicados, conversión de fechas e imputación de valores faltantes. Ante la ausencia del archivo físico de altas, se reconstruyó analíticamente la lógica de eventos de adquisición (`cliente_producto_alta.parquet`) evaluando la transición del estado de un producto de 0 a 1 entre meses consecutivos.
* **Capa Gold:** Contiene las estructuras finales listas para el consumo del modelo. Los datos se consolidaron y prepararon para las fases de entrenamiento y predicción comercial.

## 3. Desarrollo y Evaluación del Modelo
El problema se abordó como una clasificación supervisada binaria donde el modelo aprende a estimar la probabilidad de adquirir un producto candidato dado el historial del cliente.

Se entrenaron y compararon dos enfoques algorítmicos:
1. **Regresión Logística (Modelo Básico):** Utilizada como línea base de interpretación. Alcanzó un rendimiento de **AUC-ROC: 0.9639** y un **Average Precision de 0.9348**.
2. **Random Forest Classifier (Modelo Intermedio/Avanzado):** Implementado para capturar relaciones complejas y no lineales en datos tabulares. Se coronó como el modelo ganador definitivo al alcanzar métricas de **AUC-ROC: 1.0000** y **Average Precision: 1.0000** en la fase de validación.

El modelo óptimo fue exportado en formato binario (`modelo_next_best_product.pkl`) dentro de la carpeta `models/`.

## 4. Generación de Predicciones y Recomendaciones
Utilizando el modelo entrenado, se puntuó el potencial de los productos candidatos para los clientes activos. Se aplicó estrictamente la regla de negocio de excluir de las recomendaciones aquellos productos que el cliente ya posee actualmente.

El pipeline genera dos salidas clave en la capa Gold:
* `predicciones_next_best_product.parquet`: Listado de clientes, productos evaluados y sus respectivas probabilidades estimadas.
* `recomendaciones_next_best_product.parquet`: Reporte final que aísla exclusivamente el producto con la máxima probabilidad (Ranking #1) para cada cliente, optimizando la toma de decisiones del área comercial.

---

### Estructura del Repositorio
```text
next-best-product-project/
├── data/
│   ├── raw/         <-- Archivos CSV originales
│   ├── bronze/      <-- Archivos convertidos a Parquet
│   ├── silver/      <-- Datos limpios y relacionados
│   └── gold/        <-- Datasets finales y recomendaciones (.parquet)
├── notebooks/
│   ├── 01_ingesta_raw.ipynb
│   ├── 02_convertir_bronze.ipynb
│   ├── 03_limpieza_silver.ipynb
│   ├── 04_construccion_gold.ipynb
│   ├── 05_entrenamiento_modelo.ipynb
│   └── 06_prediccion_recomendacion.ipynb
├── models/
│   └── modelo_next_best_product.pkl
└── docs/
    ├── proyecto_next_best.pdf
    └── README.md  