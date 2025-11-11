# Отчет по лабораторной работе №1  
**Изучение PostgreSQL: основы реляционных баз данных, CRUD-операции и соединения**

---

## 📌 Задание 1

### Информационный поиск

1. **Онлайн-документация PostgreSQL**  
   - Добавлена в закладки: [PostgreSQL Documentation](https://www.postgresql.org/docs/)

2. **Изучение команд `\?` и `\h`**  
   - `\?` — список команд psql (управление сеансом, мета-команды)  
   - `\h` — справка по SQL-командам (например, `\h CREATE INDEX`)

    sql
    SELECT c.country_name 
    FROM events e
    JOIN venues v ON e.venue_id = v.venue_id
    JOIN countries c ON v.country_code = c.country_code
    WHERE e.title = 'Fight Club';

3. **Ограничения `MATCH`**  
   - Изучены типы ограничений для внешних ключей:  
     - `MATCH FULL` — оба столбца внешнего ключа должны быть NULL или оба не NULL  
     - `MATCH SIMPLE` (по умолчанию) — допустимы частичные NULL  
     - `MATCH PARTIAL` — устаревший, не рекомендуется

    sql
     ALTER TABLE venues ADD COLUMN active BOOLEAN DEFAULT TRUE;

##  Задание 2

### Информационный поиск

1. **Агрегатные функции PostgreSQL**  
   Изучены основные агрегатные функции:
   - `COUNT()` - подсчет количества строк
   - `SUM()` - сумма значений
   - `AVG()` - среднее значение
   - `MIN()/MAX()` - минимальное/максимальное значение
   - `STRING_AGG()` - объединение строк
   - `ARRAY_AGG()` - объединение в массив

   -- Проверяем текущее состояние
  - SELECT venue_id, name, active FROM venues WHERE name = 'Crystal Ballroom';

-- Выполняем "удаление"
  - DELETE FROM venues WHERE name = 'Crystal Ballroom';

-- Проверяем, что запись не удалена, а только деактивирована
  - SELECT venue_id, name, active FROM venues WHERE name = 'Crystal Ballroom';

2. **Графические интерфейсы для PostgreSQL**  
   Ознакомились с популярными GUI-инструментами:
   - **pgAdmin** - официальный инструмент с полной функциональностью
   - **DataGrip** - кроссплатформенная IDE от JetBrains
   - **DBeaver** - универсальный инструмент с открытым исходным кодом
   - **Navicat** - коммерческий инструмент с расширенными возможностями

  **Исходный вариант с временной таблицей**
  - CREATE TEMPORARY TABLE month_count(month INT);
  - INSERT INTO month_count VALUES (1), (2), (3), (4), (5), (6), (7), (8), (9), (10), (11), (12);

  **Улучшенный вариант с generate_series()**
  - SELECT * FROM crosstab(
    - 'SELECT EXTRACT(YEAR FROM starts) AS year,
            - EXTRACT(MONTH FROM starts) AS month,
            - COUNT(*)
     - FROM events
     - GROUP BY year, month
     - ORDER BY year, month',
    - 'SELECT generate_series(1, 12)'
  - ) AS (  - 
    - year INT,
    - jan INT, feb INT, mar INT, apr INT, may INT, jun INT,
    - jul INT, aug INT, sep INT, oct INT, nov INT, dec INT
  - )
  - ORDER BY year;

### Практические задачи

1. **Создание правила для мягкого удаления мест проведения**  
   ```sql
   CREATE OR REPLACE RULE soft_delete_venue AS 
   ON DELETE TO venues DO INSTEAD 
   UPDATE venues SET active = FALSE WHERE venue_id = OLD.venue_id;

   WITH calendar_days AS (
  SELECT 
    date::date,
    EXTRACT(DOW FROM date) as day_of_week,
    EXTRACT(WEEK FROM date) - EXTRACT(WEEK FROM date_trunc('month', date)) + 1 as week_of_month
  FROM generate_series(
    '2018-01-01'::date,
    '2018-01-31'::date,
    '1 day'::interval
  ) AS date
  ),
    - event_counts AS (
      - SELECT 
        - starts::date as event_date,
        - COUNT(*) as event_count
      - FROM events
      - GROUP BY starts::date
    - )
    - SELECT 
      - week_of_month,
      - MAX(CASE WHEN day_of_week = 0 THEN COALESCE(event_count, 0) || ' events' ELSE '' END) as sunday,
      - MAX(CASE WHEN day_of_week = 1 THEN COALESCE(event_count, 0) || ' events' ELSE '' END) as monday,
      - MAX(CASE WHEN day_of_week = 2 THEN COALESCE(event_count, 0) || ' events' ELSE '' END) as tuesday,
      - MAX(CASE WHEN day_of_week = 3 THEN COALESCE(event_count, 0) || ' events' ELSE '' END) as wednesday,
      - MAX(CASE WHEN day_of_week = 4 THEN COALESCE(event_count, 0) || ' events' ELSE '' END) as thursday,
      - MAX(CASE WHEN day_of_week = 5 THEN COALESCE(event_count, 0) || ' events' ELSE '' END) as friday,
      - MAX(CASE WHEN day_of_week = 6 THEN COALESCE(event_count, 0) || ' events' ELSE '' END) as saturday
    - FROM calendar_days cd
    - LEFT JOIN event_counts ec ON cd.date = ec.event_date
    - GROUP BY week_of_month
    - ORDER BY week_of_month;

end