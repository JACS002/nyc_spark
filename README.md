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

---

## 🧪 Análisis de resultados (2015–2025): síntesis

**Nota sobre visualización:**  
El notebook imprime en pantalla solo las primeras 20 filas (`head(20)`), por lo que muchos ejemplos muestran meses iniciales de 2015.  
Los resultados completos 2015–2025 están guardados como CSV en `evidence/analysis_05/`.

---

### 🔍 Hallazgos clave por tema

#### (a) & (b) Zonas top de pickup/dropoff  
**Archivos:** `a_top10_pickup_por_mes.csv`, `b_top10_dropoff_por_mes.csv`  
Manhattan domina el volumen mensual con clústers estables: Upper East Side (N/S), Midtown Center, Times Sq/Theatre District, Union Sq, Murray Hill.  
El patrón se repite en pickup y dropoff, consistente con zonas de alta densidad laboral y turística.

---

#### (c) Evolución mensual de `total_amount` y `tip_pct` por borough  
**Archivo:** `c_evol_total_y_tip_por_borough.csv`  
Manhattan concentra tickets promedio más estables y `tip_pct` mayores; Queens y Brooklyn muestran tickets más altos en trayectos largos (aeropuertos y viajes inter-borough).  
EWR aparece con tickets muy altos y varianza en propinas (outliers por viajes largos y peajes).

---

#### (d) Ticket promedio por servicio y mes  
**Archivo:** `d_ticket_promedio_por_service_mes.csv`  
`Yellow > Green` de forma consistente. La brecha es estable y se amplifica en meses con mayor congestión o peajes.

---

#### (e) Picos por hora y día de semana  
**Archivo:** `e_trips_por_hora_y_dow.csv`  
Picos marcados en **commute** (AM: 8–10, PM: 17–20) y actividad nocturna de fin de semana.  
El patrón es estable a lo largo de los años.

---

#### (f) p50/p90 de duración por borough  
**Archivo:** `f_p50_p90_duracion_por_borough.csv`  
Manhattan presenta p50 ≈ 10–11 min y p90 ≈ 25 min; Queens/Bronx exhiben p90 más altos por trayectos largos.

---

#### (g) Velocidad por franjas 06–09 y 17–20  
**Archivo:** `g_speed_por_franja_y_borough.csv`  
Manhattan cae a ~10–12 mph en horas pico; Queens mantiene velocidades más altas por tramos de autopistas.  
Señal clara de congestión recurrente.

---

#### (h) Participación por método de pago y `tip_pct`  
**Archivo:** `h_share_pago_y_tip.csv`  
Tarjeta domina y presenta `tip_pct` significativamente mayor vs efectivo (donde la propina tiende a cero).  
Aparece **Flex Fare** en años recientes con dinámica propia.

---

#### (i) Rate codes y su contribución a distancia/total  
**Archivo:** `i_ratecode_dist_y_total.csv`  
`Standard rate` concentra la mayor parte de distancia y total; **JFK** y **Newark** capturan parte relevante por viajes de aeropuerto (largos y con peajes).

---

#### (j) Mix yellow vs green por mes y borough  
**Archivo:** `j_mix_service_por_mes_borough.csv`  
Manhattan es abrumadoramente **yellow**, mientras Brooklyn/Queens muestran mayor **green** (especialmente en 2015–2016).  
El mix refleja reglas operativas históricas.

---

#### (k) Top 20 flujos PU→DO  
**Archivo:** `k_top20_flujos_pu_do.csv`  
Flujos intra-Manhattan de alta densidad (p. ej., Upper East ↔ Midtown / Times Sq) y algunos nodos de transferencia (Penn Station / Times Sq).  
El `AVG_TICKET` es moderado por ser distancias cortas.

---

#### (l) Pasajeros y ticket  
**Archivo:** `l_dist_passenger_y_ticket.csv`  
La moda es 1 pasajero. El ticket promedio crece suavemente hasta 3–4 pax y tiene outliers raros a recuentos atípicos (valores espurios conservados por trazabilidad).

---

#### (m) Impacto de peajes y congestión por zona  
**Archivo:** `m_impacto_tolls_congestion_por_zona.csv`  
Zonas de Manhattan muestran `congestion_surcharge` promedio alto; aeropuertos y Staten Island resaltan en `tolls_amount`.

---

#### (n) Proporción de viajes cortos vs largos  
**Archivo:** `n_short_long_por_borough_mes.csv`  
Manhattan concentra `SHORT < 1mi` y `MED < 5mi`; Queens y Bronx elevan `LONG ≥ 5mi` por geografía y aeropuertos.  
Estacionalidad moderada.

---

#### (o) Diferencias por vendor  
**Archivo:** `o_vendor_speed_y_duracion.csv`  
**Curb** y **Creative Mobile** concentran la mayoría de viajes; pequeñas diferencias en `avg_speed` y duración, consistentes con mezcla territorial.

---

#### (p) Pago ↔ tip por hora  
**Archivo:** `p_tip_por_pago_y_hora.csv`  
Con tarjeta se observan propinas positivas y estables; con efectivo, cercanas a cero.  
La estacionalidad horaria es suave.

---

#### (q) Zonas con p99 de duración/distancia altos  
**Archivo:** `q_p99_outliers_por_zona.csv`  
Outliers en **Staten Island**, **Far Rockaway / Rockaways** y **Coney Island / Marine Park**: distancias largas y/o vías lentas → potencial congestión o eventos.

---

#### (r) Yield por milla  
**Archivo:** `r_yield_por_borough_y_hora.csv`  
Yield más alto fuera de horas pico y en boroughs con menor congestión; en Manhattan cae durante 17–20.

---

#### (s) YoY volumen y ticket por servicio  
**Archivo:** `s_yoy_vol_y_ticket_por_service.csv`  
Desde 2016 ya hay comparables: se observan descensos YoY en **green** respecto a 2015, y variaciones moderadas en ticket (impacto de recargos, oferta, demanda).

---

#### (t) Días de alta congestión  
**Archivo:** `t_impacto_congestion_dias_altos.csv`  
Los días etiquetados como **ALTO_CONG** presentan `AVG_TOTAL` mayor que los días **NORMAL**, consistente con el efecto de congestión sobre el ticket.

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
