# ДЗ 3. Завантаження даних та основи SQL. DQL-команди

## Датасет

- **Назва:** Adult (Census Income), UCI Machine Learning Repository, id = 2
- **Джерело:** https://archive.ics.uci.edu/dataset/2/adult
- **Вибірка:** повний набір без семплювання: train (`adult.data`, 32 561) + test (`adult.test`, 16 281)
- **Кількість рядків:** 48 842, 15 колонок (14 ознак + `income`)

## Отримання даних

```python
from ucimlrepo import fetch_ucirepo
dataset = fetch_ucirepo(id=2)
df = pd.concat([dataset.data.features, dataset.data.targets], axis=1)
df.to_csv("data/adult.csv", index=False)
```

Notebook завантажує дані сам і зберігає їх у `data/adult.csv`.

## Таблиці

- `hw3_adult_staging` — сирі дані, усі колонки `TEXT`
- `hw3_adult_clean` — типізована таблиця з `PRIMARY KEY`, `NOT NULL` і `CHECK`

## Запуск

Відкрити `hw3.ipynb` у Google Colab і виконати `Runtime → Restart and run all`. Перша клітинка встановлює залежності, запускає PostgreSQL через `pgserver` і виводить версію.
