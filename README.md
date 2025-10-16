# 🚌 Proyecto 03 – Data Mining 2025  
### Universidad San Francisco de Quito  
**Curso:** Data Mining  
**Estudiante:** Joel Cuascota
**Fecha:** Octubre 2025  

---

## 🧠 Resumen
Este proyecto replica un flujo completo de *Data Engineering + Analytics* sobre el dataset **NYC TLC Trips (Yellow & Green, 2015–2025)**.  
La infraestructura se levanta con **Docker Compose** (servicio único `spark-notebook`) y el destino analítico es **Snowflake**, organizado en dos esquemas:

- **RAW:** aterrizaje espejo de Parquet con metadatos de ingesta.  
- **ANALYTICS:** tabla unificada `analytics.obt_trips` (modelo *One Big Table*, OBT).

Se procesan, validan y analizan los datos mediante notebooks en Jupyter/Spark o Snowpark Python.

---

## 🎯 Objetivos de aprendizaje
- Operar Spark en Jupyter para ingesta masiva y transformación ligera de Parquet.  
- Diseñar un aterrizaje RAW y una OBT desnormalizada para analítica directa.  
- Practicar un modelo alternativo al dimensional (One Big Table).  
- Gestionar seguridad y reproducibilidad con Docker y variables de ambiente.  
- Implementar controles de calidad, idempotencia y auditoría de cargas.


## 🏗️ Arquitectura general

```text
Parquet (NYC TLC 2015–2025)
   │
   ▼
Spark (Jupyter Notebook en contenedor)
   │
   ├── Ingesta RAW → Snowflake.RAW
   ├── Enriquecimiento/Unificación (zones, catálogos)
   ├── Construcción OBT (derivadas + metadatos)
   └── Validaciones y Análisis (Snowpark)
         ▼
         Snowflake.ANALYTICS.OBT_TRIPS
```

## 🧰 Herramientas clave

- **Docker + Docker Compose**  
- **Jupyter + PySpark**  
- **Snowflake (con Snowpark Python)**  
- **Pandas + Matplotlib** para visualizaciones  

---

## ⚙️ Variables de ambiente (`.env`)

Todas las credenciales y parámetros se manejan mediante un archivo `.env`  
*(sin credenciales reales en GitHub)*.

### Ejemplo de `.env.example`
```bash
SNOWFLAKE_ACCOUNT=xxxxxx
SNOWFLAKE_USER=xxxxx
SNOWFLAKE_PASSWORD=xxxxx
SNOWFLAKE_ROLE=ACCOUNTADMIN
SNOWFLAKE_WAREHOUSE=COMPUTE_WH
SNOWFLAKE_DATABASE=NYC_TAXI_DM
SNOWFLAKE_SCHEMA_RAW=RAW
SNOWFLAKE_SCHEMA_ANALYTICS=ANALYTICS
PARQUET_PATH=/data/parquet
RUN_ID=P3_$(date +%Y%m%d_%H%M)
```

## 🧩 Notebooks y propósito

| Notebook | Propósito principal |
|-----------|--------------------|
| **01_ingesta_parquet_raw.ipynb** | Lee Parquet (Yellow/Green) 2015–2025 y carga a RAW. |
| **02_enriquecimiento_y_unificacion.ipynb** | Integra catálogos (zones, vendor, rate, payment). |
| **03_construccion_obt.ipynb** | Construye `analytics.obt_trips` (derivadas, idempotencia). |
| **04_validaciones_y_exploracion.ipynb** | Valida nulos, rangos, coherencia, conteos. |
| **05_data_analysis.ipynb** | Responde 20 preguntas de negocio con `analytics.obt_trips`. |


## 🗓️ Cobertura procesada

La siguiente matriz resume la cobertura temporal y el estado de procesamiento por servicio.  
Los resultados completos se documentan en el archivo CSV de evidencia:

📄 **`evidence/matriz_cobertura.csv`**

## 📦 Diseño de esquemas

### 🗂️ RAW
- **Grano:** viaje original.  
- **Metadatos:** `_run_id`, `_ingested_at_utc`, `service`, `year`, `month`.  
- **Uso:** staging y auditoría.  

---

### 🧮 ANALYTICS.OBT_TRIPS (One Big Table)
- **Grano:** 1 fila = 1 viaje.  
- **Derivadas:**
  - `trip_duration_min = datediff('minute', pickup, dropoff)`  
  - `avg_speed_mph = distance / (duration/60)`  
  - `tip_pct = tip_amount / fare_amount`  
- **Metadatos:** `run_id`, `built_at_utc`, `source_service`, `source_year`, `source_month`.  
- **Idempotencia:** control mediante `TRIP_ID = HASH(...)`.

---

## ✅ Calidad y auditoría

**Validaciones ejecutadas:**
- Nulos en campos esenciales (`pickup`, `dropoff`, `location`, `payment`) → ✅ 0 nulos.  
- **Rangos lógicos:**
  - Duración: 0–48h  
  - Distancia: 0–150mi  
  - Velocidad ≤ 100 mph → ✅ sin fuera de rango.  
- **Coherencia temporal:** `dropoff ≥ pickup` → ✅.  
- **Conteos por servicio/mes:** reproducen los conteos originales (12.7M yellow, 1.5M green).  

**Auditoría:**  
Archivo `validacion_obt_quality_summary.csv` con métricas por servicio.


## 📊 Resultados analíticos (enero 2015)

| Indicador | Hallazgo |
|------------|-----------|
| **Top zonas pickup/dropoff** | Manhattan (UES, Midtown, Times Sq) dominan. |
| **Ticket promedio** | \$15 (yellow), \$14.7 (green). |
| **Tip promedio** | ~14% en tarjeta. |
| **Duración p50/p90** | 10 / 22 min en Manhattan. |
| **Velocidad promedio** | 12–13 mph, cae en hora pico. |
| **RateCode top** | Standard, JFK, Newark. |
| **Mix service** | 96% yellow en Manhattan, 70% green en Brooklyn. |
| **Métodos de pago** | 60% tarjeta, 39% efectivo. |
| **YoY** | Sin datos previos (solo enero 2015). |

---

## 🧱 Ejecución paso a paso

1. Clonar el repositorio y crear el archivo `.env` (basado en `.env.example`).  
2. Levantar el entorno con Docker Compose:  
   ```bash
   docker compose up -d
   ```
3. Acceder a Jupyter: http://localhost:8888
4. Ejecutar los notebooks en orden: 01 → 05
5. Revisar los outputs en las carpetas: evidence/


## 🗂️ Carpeta de evidencias (`/evidence`)

| Evidencia               | Descripción esperada                                               |
|-------------------------|-------------------------------------------------------------------|
| `docker_running.png`     | `docker ps` mostrando contenedor `spark-notebook` activo.        |
| `jupyter_home.png`       | Vista de JupyterLab con notebooks visibles.                      |
| `spark_ui.png`           | Spark UI (puerto 4040) ejecutando tareas.                        |
| `raw_counts.png`         | Conteos por servicio/año/mes después de ingesta RAW.             |
| `obt_summary.png`        | Conteo total en `analytics.obt_trips` (12.7M yellow, 1.5M green). |
| `validaciones.png`       | Resultados de `04_validaciones_y_exploracion.ipynb`.             |
| `analysis_outputs.png`   | Muestra de resultados (top zonas, payment, vendor, etc.).        |
| `snowflake_console.png`  | Vista de las tablas RAW y OBT en Snowflake.                      |


## 🧾 Checklist de aceptación

- Docker Compose levanta Spark + Jupyter
- Variables desde `.env`
- Carga completa 2015–2025 (al menos validado 2015–01)
- `analytics.obt_trips` creada con derivadas y metadatos
- Idempotencia verificada (reingesta 2015–01)
- Validaciones completas (rangos, nulos, coherencia)
- 20 preguntas analizadas
- README con pasos y evidencias

---

## 📈 Conclusiones

- Se logró una OBT robusta, idempotente y sin duplicados.
- Los resultados analíticos confirman patrones históricos de tráfico en NYC.
- Snowpark resultó más simple y eficiente que Spark puro para análisis SQL.
- Infraestructura reproducible vía Docker y variables de entorno.
