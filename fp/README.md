# Фінальний проєкт. Реляційні бази даних

## Дані

Датасет [infectious_cases.csv](https://drive.google.com/file/d/1dQf-uVXzYPYYGgXsG1y4Q1qDdhP2miZl/view): 10 521 рядок, статистика інфекційних захворювань за 245 країнами та регіонами, 1920–2023 роки.
Notebook сам завантажує файл з Google Drive через `gdown` у `data/infectious_cases.csv`.

## Виконання

1. **Завантаження.** PostgreSQL запускається через `pgserver`. Створюється схема `pandemic` (`CREATE SCHEMA`, `SET search_path`), таблиця `infectious_cases` з колонками `TEXT`, дані імпортуються через `COPY FROM STDIN`.
2. **Нормалізація до 3NF.** `code` залежить лише від `entity`, а не від усього ключа `(entity, year)`, тому дані розбито на дві таблиці:
   - `entities (id, entity, code)` — 245 унікальних пар `entity` + `code`;
   - `infectious_cases_normalized (entity_id, year, ...)` — показники з `entity_id` замість `entity` і `code`, ключ `(entity_id, year)`, типи `INTEGER` / `NUMERIC`.

   Перевірка через `EXCEPT` підтверджує, що з нормалізованих таблиць відновлюється оригінал.
3. **Аналіз `number_rabies`.** `AVG`, `MIN`, `MAX`, `SUM` для кожної `entity`, сортування за середнім за спаданням, 10 рядків.
4. **Різниця в роках.** `MAKE_DATE(year, 1, 1)`, `CURRENT_DATE` і `EXTRACT(YEAR FROM AGE(...))`.
5. **Функції.**
   - `fn_years_since(year)` — різниця в роках між поточною датою та 1 січня заданого року.
   - `fn_cases_per_period(cases, divisor)` — середня кількість захворювань за період; перевірено з дільниками 12, 4 і 2 на рядках, де `number_rabies` є числом.

## Запуск

Відкрити `fp.ipynb` у Google Colab і виконати `Runtime → Restart and run all`.
