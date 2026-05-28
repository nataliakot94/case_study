# 🎓 Решение аналитических задач по SQL (База данных Skyeng)


В этом блоке собраны запросы для анализа логов проведенных уроков в таблице `skyeng_db.classes`. Основной фокус задач — работа с датами, интервалами времени (`INTERVAL`), расчет длительности занятий и сегментация данных.

---

### 📑 Задание 1. Анализ длительности регулярных уроков за 2016 год
**Бизнес-задача:** Выгрузить успешные регулярные уроки за 2016 год, рассчитать их чистую продолжительность и разметить флажком `flag_len` занятия, которые длились строго более 1 часа 30 минут.

```sql
SELECT 
    id_class, 
    class_start_datetime, 
    class_end_datetime, 
    class_end_datetime - class_start_datetime AS len_lesson,
    CASE 
        WHEN class_end_datetime - class_start_datetime > INTERVAL '1 hours 30 minutes' THEN 1 
        ELSE 0 
    END AS flag_len
FROM skyeng_db.classes
WHERE EXTRACT('year' FROM class_start_datetime) = 2016 
  AND class_end_datetime - class_start_datetime > INTERVAL '0 seconds'
  AND class_status = 'success'
  AND class_type = 'regular'
ORDER BY class_start_datetime ASC;
```

---

### 📑 Задание 2. Топ-30 самых долгих успешных уроков в I квартале
**Бизнес-задача:** Найти 30 самых продолжительных успешных уроков, прошедших в период с 1 января по 1 апреля 2016 года, для выявления возможных аномалий или проведения длительных интенсивов.

```sql
SELECT 
    id_class, 
    user_id, 
    id_teacher, 
    class_start_datetime, 
    class_end_datetime, 
    class_end_datetime - class_start_datetime AS len_lesson
FROM skyeng_db.classes
WHERE class_end_datetime > class_start_datetime
  AND class_status = 'success'
  AND class_start_datetime BETWEEN '2016-01-01' AND '2016-04-01'
ORDER BY len_lesson DESC
LIMIT 30;
```

---

### 📑 Задание 3. Поиск аномальных по длительности занятий (Апрель 2016)
**Бизнес-задача:** Отфильтровать нестандартные уроки за апрель 2016 года: выделить слишком длинные (от 2.5 часов) и слишком короткие (до 20 минут) кейсы. Дополнительно разметить статус успешности урока числовым флагом.

```sql
SELECT 
    id_class, 
    class_start_datetime, 
    class_end_datetime, 
    class_end_datetime - class_start_datetime AS len_lesson,
    CASE 
        WHEN (class_end_datetime - class_start_datetime) >= INTERVAL '2 hour 30 minute' THEN 'Слишком длинный'
        WHEN (class_end_datetime - class_start_datetime) <= INTERVAL '20 minute' THEN 'Слишком короткий'
        ELSE 'Нормальный' 
    END AS flag_duration,
    CASE 
        WHEN class_status = 'success' THEN 1 
        ELSE 0 
    END AS flag_status
FROM skyeng_db.classes
WHERE class_end_datetime > class_start_datetime 
  AND (
       (class_end_datetime - class_start_datetime > '2 hours 30 mins') 
    OR (class_end_datetime - class_start_datetime < '0 hours 20 mins')
  )
  AND class_start_datetime BETWEEN '2016-04-01' AND '2016-05-01'
ORDER BY class_start_datetime
LIMIT 10;
```

---

### 📑 Задание 4. Анализ пиковых часов нагрузки (Утро / Вечер)
**Бизнес-задача:** Категоризировать уроки 2016 года по времени начала на «Утренние» (с 7 до 9 утра) и «Вечерние» (со 17 до 19 вечера). Исключить из выгрузки первые числа месяцев (`day <> 1`) для очистки от возможных системных авто-списаний.

```sql
SELECT 
    id_class, 
    class_start_datetime, 
    class_type,  
    CASE 
        WHEN EXTRACT('hour' FROM class_start_datetime) BETWEEN 7 AND 9 THEN 'Утро'
        WHEN EXTRACT('hour' FROM class_start_datetime) BETWEEN 17 AND 19 THEN 'Вечер'
        ELSE 'Другое' 
    END AS lesson_time
FROM skyeng_db.classes
WHERE class_end_datetime > class_start_datetime 
  AND EXTRACT('year' FROM class_start_datetime) = 2016
  AND DATE_PART('day', class_start_datetime) <> 1
  AND (
       (EXTRACT('hour' FROM class_start_datetime) BETWEEN 7 AND 9)
    OR (EXTRACT('hour' FROM class_start_datetime) BETWEEN 17 AND 19)
  )
  AND class_status IS NOT NULL 
ORDER BY class_start_datetime
LIMIT 100;
```
