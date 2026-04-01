#  Documentación Capa Gold – ETL Riesgos Agronómicos

Esta documentación describe las tablas generadas por el proceso **ETL Gold** (`ETL-Gold.ipynb`) en Databricks utilizando **Delta Tables**.

---

##  Tablas generadas

* `daily_actual_gold`
* `daily_forecast_gold`
* `astronomy_daily_gold`
* `agricultural_risk_metrics`

 **Fuentes Silver utilizadas:**

* `weather_daily_silver`
* `weather_daily_silver_forecast`
* `weather_hourly_silver`
* `weather_astronomy_daily_silver`

---

#  daily_actual_gold

##  Descripción

Resumen diario histórico por ciudad, con métricas agronómicas.

* **Granularidad:** `city + date`
* **Fuente:** `weather_daily_silver`
* **Deduplicación:** último `ingestion_time`

##  Columnas

### Base

* `city`, `date`, `ingestion_time`
* `maxtemp_c`, `mintemp_c`, `avgtemp_c`
* `avghumidity`, `totalprecip_mm`
* `maxwind_kph`, `daily_chance_of_rain`
* `condition`

### Calculadas

* `temp_range = maxtemp_c - mintemp_c`

* `frost_risk`:

  * 🔴 Alto → `mintemp_c < 2` y `maxwind_kph < 15`
  * 🟡 Moderado → `mintemp_c < 2`
  * 🟢 Bajo → resto

* `dry_day = (totalprecip_mm == 0)`

* `gdd_daily`:

```sql
max(0, ((maxtemp_c + mintemp_c)/2) - 10)
```

---

#  daily_forecast_gold

##  Descripción

Pronóstico diario con análisis de siembra.

* **Granularidad:** `city + date`
* **Fuente:** `weather_daily_silver_forecast`
* **Deduplicación:** último `ingestion_time`

##  Columnas

### Base

* `city`, `date`, `ingestion_time`
* `maxtemp_c`, `mintemp_c`, `avgtemp_c`
* `avghumidity`, `totalprecip_mm`
* `maxwind_kph`, `daily_chance_of_rain`, `uv`

### Calculadas

* `temp_range`

* `frost_risk` (misma lógica que actual)

* `sowing_window`:

  * 🟢 Óptimo → humedad > 60 y lluvia < 30 y precip < 10
  * 🟡 Marginal → humedad > 50 y lluvia < 50
  * 🔴 No apto → resto

---

#  astronomy_daily_gold

##  Descripción

Información astronómica diaria.

* **Granularidad:** `city + date`
* **Fuente:** `weather_astronomy_daily_silver`
* **Deduplicación:** último `ingestion_time`

##  Columnas

### Base

* `city`, `date`, `ingestion_time`
* `sunrise_ts`, `sunset_ts`
* `moonrise`, `moonset`
* `moon_phase`, `moon_illumination`

### Calculada

```sql
day_length_hours = (sunset_ts - sunrise_ts) / 3600
```

---

#  agricultural_risk_metrics

##  Descripción

Tabla final de riesgos lista para dashboard.

* **Granularidad:** `city + date`

##  Fuentes

* `daily_actual_gold`
* Agregados desde `weather_hourly_silver`

##  Agregaciones horarias

```sql
max_hourly_precip = max(precip_mm)
max_hourly_wind = max(wind_kph)
fungal_hours = sum(humidity > 80 AND temp_c BETWEEN 15 AND 25)
```

 **Importante:**
`fungal_hours` se calcula pero **NO se persiste actualmente**.

---

##  Columnas finales

* `city`, `date`
* `temp_range`, `frost_risk`, `dry_day`, `gdd_daily`
* `max_hourly_precip`, `max_hourly_wind`

###  storm_risk

* 🔴 Alto → precip > 10 y viento > 40
* 🟡 Moderado → precip > 5 y viento > 30
* 🟢 Bajo → resto

---

#  Cómo consumir la capa Gold

##  Uso recomendado (Dashboard)

| Caso            | Tabla                       |
| --------------- | --------------------------- |
| Clima histórico | `daily_actual_gold`         |
| Pronóstico      | `daily_forecast_gold`       |
| Riesgos         | `agricultural_risk_metrics` |
| Astronomía      | `astronomy_daily_gold`      |

---

##  Ejemplo SQL

```sql
SELECT 
  date, 
  storm_risk, 
  max_hourly_precip, 
  max_hourly_wind
FROM agricultural_risk_metrics
WHERE city = 'casilda'
ORDER BY date DESC;
```

---

## B) Reglas importantes

*  Siempre filtrar por `city` y `date`
*  No unir por `ingestion_time`
*  Unir tablas por `city + date`
*  Forecast = futuro (no mezclar con histórico sin criterio)

---

##  Riesgo fúngico

Actualmente:

* Se calcula ✔️
* No se guarda ❌

### Opciones:

* ✔️ Agregarlo a `agricultural_risk_metrics`
* ✔️ Crear tabla nueva: `hourly_aggregated_risks`

---

#  Uso en Dashboard

##  Alertas

* Heladas → `daily_actual_gold.frost_risk`
* Tormentas → `agricultural_risk_metrics.storm_risk`
* Viento → `daily_actual_gold.maxwind_kph`
* Fúngico → (recomendado persistir)

---

##  Resumen diario

* `temp_range`
* `dry_day`
* `gdd_daily`

---

##  Tendencias

* GDD → `daily_actual_gold.gdd_daily`
* Siembra → `daily_forecast_gold.sowing_window`

---

##  Calendario agrícola

* `sowing_window`
* `frost_risk`
* `storm_risk`

---

#  Conclusión

La capa **Gold** está diseñada para:

*  Consumo directo en dashboards
*  Consultas rápidas
*  Análisis agronómico

Con una estructura clara basada en:

 `city + date` como clave principal
 Métricas listas para negocio
Riesgos pre-calculados

---

