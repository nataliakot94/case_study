# 🎵 Аналитика музыкального сервиса (База данных SkyMusic)

В данном разделе представлены 6 аналитических запросов к таблице `skymusic.singer_list`. Набор задач сфокусирован на расчете агрегированных метрик (объемы, максимумы, средние значения), сегментации исполнителей по популярности и оценке долей успешных треков с помощью продвинутой фильтрации.

---

### 📑 Задание 1. Динамика регистрации исполнителей
**Бизнес-задача:** Посчитать количество артистов, добавленных на платформу в разрезе месяцев (`month_added`), зафиксированных в отчетном периоде за март 2021 года.

```sql
SELECT 
    month_added,
    COUNT(*) AS cnt
FROM skymusic.singer_list 
WHERE month_info = '2021-03-01'
GROUP BY month_added
ORDER BY month_added;
```

---

### 📑 Задание 2. Поиск максимального рейтинга артистов
**Бизнес-задача:** Определить максимальный балл/рейтинг (`score`) среди исполнителей в зависимости от месяца их появления на платформе за отчетный период марта 2021 года.

```sql
SELECT 
    month_added,
    MAX(score) AS cnt_score
FROM skymusic.singer_list 
WHERE month_info = '2021-03-01'
GROUP BY month_added
ORDER BY month_added;
```

---

### 📑 Задание 3. Средний рейтинг популярных артистов
**Бизнес-задача:** Рассчитать средний балл исполнителей, сгруппированных по месяцу добавления, отсеяв только востребованных артистов (у которых 170 и более попаданий в плейлисты).

```sql
SELECT 
    month_added,
    AVG(score) AS avg_score
FROM skymusic.singer_list 
WHERE month_info = '2021-03-01'
  AND cnt_playlist >= 170
GROUP BY month_added
ORDER BY month_added;
```

---

### 📑 Задание 4. Анализ топ-сегмента новых исполнителей
**Бизнес-задача:** Посчитать общий пул песен в топ-чартах и средний балл для относительно «новых» артистов (зарегистрированных после июня 2018 года), которые имеют либо высокий рейтинг ($\ge 8$), либо обширное присутствие в плейлистах ($\ge 200$).

```sql
SELECT 
    AVG(score) AS avg_score,
    SUM(cnt_song_top_chart) AS cnt_song_top_chart
FROM skymusic.singer_list 
WHERE month_info = '2021-03-01'
  AND (cnt_playlist >= 200 OR score >= 8)
  AND month_added >= '2018-06-01';
```

---

### 📑 Задание 5. Распределение топ-треков по округленному рейтингу
**Бизнес-задача:** Сгруппировать данные по округленному рейтингу исполнителей, посчитать суммарное количество их песен в чартах и общее число артистов. Вывести только те группы рейтинга, где присутствует более 10 исполнителей.

```sql
SELECT 
    ROUND(score) AS round_score,
    SUM(cnt_song_top_chart) AS cnt_song_top_chart1,
    COUNT(singer_id) AS count_singer_id
FROM skymusic.singer_list 
WHERE month_info = '2021-03-01'
GROUP BY round_score
HAVING COUNT(singer_id) > 10
ORDER BY round_score;
```

---

### 📑 Задание 6. Расчет долей успешных исполнителей (Расширенная аналитика)
**Бизнес-задача:** Рассчитать для каждого месяца добавления две метрики: долю артистов, имеющих более 7 песен в топ-чартах, и долю артистов с более чем 100 попаданиями в плейлисты. Включить в отчет месяцы, в которых зарегистрировано более 7 исполнителей.

```sql
SELECT 
    month_added, 
    SUM(CASE WHEN cnt_song_top_chart > 7 THEN 1.0 ELSE 0.0 END) / COUNT(*) AS share_cnt_song_top_chart, 
    SUM(CASE WHEN cnt_playlist > 100 THEN 1.0 ELSE 0.0 END) / COUNT(*) AS share_cnt_playlist
FROM skymusic.singer_list 
WHERE month_info = '2021-03-01'
GROUP BY month_added
HAVING COUNT(singer_id) > 7
ORDER BY month_added;
```
