# 🚖 Аналитика транспортной логистики (База данных SkyTaxi)

В данном разделе представлены 3 комплексных аналитических запроса к таблице логистики `skytaxi.airport_visit`. В решениях активно используются обобщенные табличные выражения (`CTE / WITH`) для декомпозиции сложных многошаговых задач, рассчитывается конверсия выезда с заказом из аэропорта и анализируется корреляция между временем ожидания водителей и пиковыми часами нагрузки.

---

### 📑 Задание 1. Анализ эффективности топ-10 водителей с максимальным временем ожидания
**Бизнес-задача:** Выделить водителей, совершивших более одного визита в аэропорт, и найти среди них топ-10 тех, кто в среднем дольше всех ждал заказ. Для этой когорты «самых терпеливых» водителей рассчитать суммарное количество успешных поездок (выезд с заказом) и общее число холостых простоев.

```sql
WITH drivers AS (
    SELECT id_driver
    FROM skytaxi.airport_visit
    GROUP BY id_driver
    HAVING COUNT(*) > 1
),
top_list_driver AS (
    SELECT 
        d.id_driver,
        AVG(time_left - time_came) AS avg_time
    FROM drivers d
    JOIN skytaxi.airport_visit av ON d.id_driver = av.id_driver
    GROUP BY d.id_driver
    ORDER BY avg_time DESC
    LIMIT 10
)
SELECT 
    SUM(CASE WHEN left_w_order = 1 THEN 1 ELSE 0 END) AS success_race,
    SUM(CASE WHEN left_w_order = 0 THEN 1 ELSE 0 END) AS race
FROM top_list_driver tld 
JOIN skytaxi.airport_visit av ON tld.id_driver = av.id_driver;
```

---

### 📑 Задание 2. Метрики эффективности на высоконагруженных узлах среди перерабатывающих водителей
**Бизнес-задача:** Проанализировать визиты в аэропорты, соответствующие двум жестким критериям: водитель суммарно провел в ожидании более 12 часов (переработка), а сам аэропорт является крупным хабом (более 100 визитов). Для отфильтрованных логов рассчитать среднее время ожидания и общую конверсию выезда с пассажиром (`flag_left_w_order`).

```sql
WITH drivers AS (
    SELECT 
        id_driver,
        SUM(time_left - time_came) AS waiting_time
    FROM skytaxi.airport_visit
    GROUP BY id_driver
    HAVING SUM(time_left - time_came) > INTERVAL '12 hour'
),
visit_airoport AS (
    SELECT 
        id_port,
        COUNT(*) AS cnt
    FROM skytaxi.airport_visit
    GROUP BY id_port
    HAVING COUNT(*) > 100
)
SELECT 
    AVG(av.time_left - av.time_came) AS avg_waiting_time,
    SUM(CASE WHEN av.left_w_order = 1 THEN 1.0 ELSE 0.0 END) / COUNT(*) AS flag_left_w_order
FROM skytaxi.airport_visit av 
WHERE av.id_driver IN (SELECT id_driver FROM drivers)
  AND av.id_port IN (SELECT id_port FROM visit_airoport);
```

---

### 📑 Задание 3. Поиск пересечения «быстрых часов» и «максимальной конверсии» в Домодедово
**Бизнес-задача:** Проверить гипотезу синхронности: совпадают ли часы, когда в аэропорту Домодедово самое короткое среднее время ожидания заказа (топ-5 быстрых часов в CTE `wait_min`), с часами, когда наблюдается максимальная вероятность уехать с пассажиром (топ-5 часов по конверсии в CTE `wait_max`). 
**Результат:** Внутреннее объединение (`JOIN`) оставляет в финальной выборке только те часы суток, где высокая эффективность совпала со скоростью обслуживания.

```sql
WITH wait_min AS (
    SELECT 
        DATE_PART('hour', time_came) AS hh,
        AVG(time_left - time_came) AS min_wait
    FROM skytaxi.airport_visit
    WHERE id_port = 'Домодедово'
    GROUP BY hh
    ORDER BY min_wait ASC
    LIMIT 5
),
wait_max AS (
    SELECT 
        DATE_PART('hour', time_came) AS hh,
        SUM(CASE WHEN left_w_order = 1 THEN 1 ELSE 0 END)::FLOAT / COUNT(*) AS sum_flag
    FROM skytaxi.airport_visit
    WHERE id_port = 'Домодедово'
    GROUP BY hh
    ORDER BY sum_flag DESC
    LIMIT 5
)
SELECT 
    w.hh,
    w.min_wait,
    m.sum_flag
FROM wait_min w 
JOIN wait_max m ON w.hh = m.hh
ORDER BY w.hh;
```
