# ДЗ 7. Time-series features і JSONB у PostgreSQL

## Дані

- **NYC TLC Yellow Taxi Trip Records** — https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
  - https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-02.parquet
  - https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-03.parquet
- **JSONB event-log** — синтетичний, генерується SQL-запитом у notebook (`generate_series`, 50 000 подій).

Stub-schema fallback не використовується.

## Отримання даних

Notebook сам завантажує обидва Parquet-файли з офіційного CDN TLC (у `/content` в Colab або в `data/` локально).
З кожного місяця береться випадкова вибірка 250 000 поїздок (`random_state=42`), разом 500 000.
Два місяці замість одного, бо `head(500_000)` з одного файлу покриває лише 6 днів, і `trips_lag_30d` був би порожнім.

Базове очищення: `0 < fare_amount < 500`, `0 < trip_distance < 100`, `PULocationID IS NOT NULL`, pickup у межах 2024-02-01 — 2024-03-31.
Результат завантажується в `nyc_taxi_raw`, далі агрегується в `t7_hw_taxi_daily` (8 486 рядків, 240 зон, 60 днів).

## Time-series features

Усі вікна мають `PARTITION BY pickup_zone_id`.

| Feature | Window |
|---|---|
| `trips_lag_1d`, `trips_lag_7d`, `trips_lag_30d` | `LAG` по `ts_day` |
| `rolling_avg_fare_7d` | `RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW` |
| `rolling_std_fare_30d` | `STDDEV_SAMP`, `RANGE BETWEEN INTERVAL '29 days' PRECEDING AND CURRENT ROW` |
| `cumulative_trips`, `expanding_avg_trips` | `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` |

Усі ознаки зібрані у view `v_t7_hw_taxi_features` (рядки з наявним `trips_lag_30d`).

## JSONB

- `t7_hw_events` — 5 типів подій з різною структурою `payload`.
- Запити: containment `@>`, key existence `?` / `?|`, JSONPath `jsonb_path_query` / `jsonb_path_exists`.
- `EXPLAIN (ANALYZE, BUFFERS)` до і після GIN (`jsonb_path_ops`) та до і після expression B-tree на `payload ->> 'device'`.
- `t7_hw_checkouts_norm` — нормалізована таблиця checkout-подій і порівняння одного запиту в JSONB- та normalized-версіях.
- MongoDB-equivalent як pseudocode у markdown.

## Запуск

1. Відкрити `hw7.ipynb` у Google Colab.
2. Runtime → Restart and run all.

Перша клітинка встановлює залежності (`pgserver`, `jupysql`, `psycopg2-binary`, `sqlalchemy`, `pandas`, `pyarrow`), запускає PostgreSQL у `/tmp/hw7_pg` і виводить його версію.
