# Capas Gold: weather & astronomy

Esta capa Gold consolida la información diaria y horaria de clima y datos astronómicos, combinando las tablas Silver de las carpetas **raw/astronomy**, **forecast/weather** e **history/weather**.  
Se genera un nivel de granularidad adecuado para análisis diarios y horarios, agregando métricas relevantes para cada ciudad y fecha.

---

## Tabla: weather_daily_gold
📌 **Descripción**  
Resumen diario histórico de clima por ciudad y fecha, a partir de `weather_daily_silver` (History).  
Cada registro representa:

- 1 ciudad
- 1 día (fecha del registro)
- Último registro disponible según `ingestion_time`

📊 **Granularidad**  
`city + date`

🧱 **Columnas**

| Columna | Tipo | Descripción |
|---------|------|------------|
| city | string | Nombre de la ciudad |
| date | date | Fecha del registro |
| ingestion_time | timestamp | Momento de ingesta del registro |
| maxtemp_c | double | Temperatura máxima del día (°C) |
| mintemp_c | double | Temperatura mínima del día (°C) |
| avgtemp_c | double | Temperatura promedio del día (°C) |
| avghumidity | bigint | Humedad promedio (%) |
| totalprecip_mm | double | Precipitación total (mm) |
| maxwind_kph | double | Velocidad máxima del viento (km/h) |
| daily_chance_of_rain | bigint | Probabilidad de lluvia (%) |
| condition | string | Condición general del día (texto) |

⚙️ **Lógica aplicada**  
- Se elimina duplicados usando ventana: `partitionBy(city, date).orderBy(ingestion_time DESC)`  
- Se queda con el último registro diario por ciudad

---

## Tabla: weather_hourly_gold
📌 **Descripción**  
Resumen horario histórico de clima por ciudad y fecha, a partir de `weather_hourly_silver`.  
Cada registro representa:

- 1 ciudad
- 1 hora específica del día

📊 **Granularidad**  
`city + date + datetime`

🧱 **Columnas**

| Columna | Tipo | Descripción |
|---------|------|------------|
| city | string | Nombre de la ciudad |
| date | date | Fecha del registro |
| datetime | timestamp | Fecha y hora exacta del registro |
| temp_c | double | Temperatura en °C |
| humidity | bigint | Humedad (%) |
| precip_mm | double | Precipitación en mm |
| wind_kph | double | Velocidad del viento (km/h) |
| cloud | bigint | Porcentaje de nubosidad |
| condition | string | Condición del clima (texto) |
| ingestion_time | timestamp | Momento de ingesta del registro |

⚙️ **Lógica aplicada**  
- Se eliminan duplicados por `city + datetime` tomando el último `ingestion_time`

---

## Tabla: weather_daily_forecast_gold
📌 **Descripción**  
Resumen diario forecast de clima por ciudad y fecha, a partir de `weather_daily_silver_forecast`.  
Cada registro representa:

- 1 ciudad
- 1 día predicho
- Último forecast disponible

📊 **Granularidad**  
`city + date`

🧱 **Columnas**

| Columna | Tipo | Descripción |
|---------|------|------------|
| city | string | Nombre de la ciudad |
| date | date | Fecha del forecast |
| ingestion_time | timestamp | Momento de ingesta del forecast |
| maxtemp_c | double | Temperatura máxima prevista (°C) |
| mintemp_c | double | Temperatura mínima prevista (°C) |
| avgtemp_c | double | Temperatura promedio prevista (°C) |
| avghumidity | bigint | Humedad promedio prevista (%) |
| totalprecip_mm | double | Precipitación total prevista (mm) |
| maxwind_kph | double | Velocidad máxima del viento prevista (km/h) |
| daily_chance_of_rain | bigint | Probabilidad de lluvia (%) |
| uv | double | Índice UV previsto |

⚙️ **Lógica aplicada**  
- Se elimina duplicados por `city + date` tomando último `ingestion_time`  
- Se queda con 1 fila diaria por ciudad

---

## Tabla: weather_hourly_forecast_gold
📌 **Descripción**  
Resumen horario forecast por ciudad y fecha, a partir de `weather_hourly_silver_forecast`.  
Cada registro representa:

- 1 ciudad
- 1 hora predicha

📊 **Granularidad**  
`city + date + time`

🧱 **Columnas**

| Columna | Tipo | Descripción |
|---------|------|------------|
| city | string | Nombre de la ciudad |
| date | date | Fecha del forecast |
| time | timestamp | Hora específica del forecast |
| hour | int | Hora del día (0-23) |
| temp_c | double | Temperatura prevista (°C) |
| feelslike_c | double | Temperatura percibida (°C) |
| uv | double | Índice UV |
| precip_mm | double | Precipitación prevista (mm) |
| humidity | bigint | Humedad prevista (%) |
| wind_kph | double | Velocidad del viento prevista (km/h) |
| cloud | bigint | Nubosidad prevista (%) |
| chance_of_rain | bigint | Probabilidad de lluvia (%) |
| will_it_rain | bigint | Indicador binario si lloverá (0/1) |
| ingestion_time | timestamp | Momento de ingesta del forecast |

⚙️ **Lógica aplicada**  
- Se elimina duplicados por `city + time` tomando último `ingestion_time`

---

## Tabla: weather_astronomy_gold
📌 **Descripción**  
Resumen diario astronómico por ciudad y fecha, a partir de `weather_astronomy_daily_silver`.  
Cada registro representa:

- 1 ciudad
- 1 día
- Últimos datos astronómicos disponibles

📊 **Granularidad**  
`city + date`

🧱 **Columnas**

| Columna | Tipo | Descripción |
|---------|------|------------|
| city | string | Nombre de la ciudad |
| date | date | Fecha del registro |
| sunrise_ts | timestamp | Hora exacta de amanecer |
| sunset_ts | timestamp | Hora exacta de atardecer |
| moonrise | string | Hora de salida de la Luna |
| moonset | string | Hora de puesta de la Luna |
| moon_phase | string | Fase lunar (texto) |
| moon_illumination | bigint | Iluminación lunar (%) |
| ingestion_time | timestamp | Momento de ingesta del registro |

⚙️ **Lógica aplicada**  
- Se elimina duplicados por `city + date` tomando último `ingestion_time`  
- Se mantienen `sunrise_ts` y `sunset_ts` para cálculos posteriores (duración del día, comparaciones interdiarias)
