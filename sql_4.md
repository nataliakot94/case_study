# 🎬 Аналитика кинематографа (База данных IMDb)

В данном разделе представлены 4 аналитических запроса к связанным таблицам `imdb.title_basics` и `imdb.title_ratings`. Задачи демонстрируют продвинутую работу со всеми типами объединений (`INNER`, `LEFT`, `RIGHT JOIN`), агрегацию временных интервалов, бинаризацию непрерывных величин и аудит целостности данных.

---

### 📑 Задание 1. Суммарный хронометраж лучших анимационных фильмов
**Бизнес-задача:** Посчитать общее время просмотра (в часах) для всех анимационных фильмов, выпущенных с 1990 года, которые получили высокую оценку зрителей (рейтинг $> 8$).

```sql
SELECT 
    SUM("runtimeMinutes") / 60 AS total_hours
FROM imdb.title_basics a
JOIN imdb.title_ratings b ON a.tconst = b.tconst
WHERE genres LIKE '%Animation%'
  AND "startYear" >= 1990
  AND "averageRating" > 8;
```

---

### 📑 Задание 2. Расчет доли фильмов без пользовательских оценок
**Бизнес-задача:** Оценить полноту данных на платформе. Найти долю полнометражных фильмов (от 60 минут), выпущенных после 1950 года, которые не имеют ни одного рейтинга от пользователей (`NULL`).

```sql
SELECT 
    SUM(CASE WHEN "averageRating" IS NULL THEN 1.0 ELSE 0.0 END) / COUNT(*) AS share_unrated_titles
FROM imdb.title_basics a
LEFT JOIN imdb.title_ratings b ON a.tconst = b.tconst
WHERE "startYear" >= 1950
  AND "runtimeMinutes" >= 60;
```

---

### 📑 Задание 3. Распределение фильмов «для взрослых» по интервалам рейтинга
**Бизнес-задача:** Категоризировать (разбить на бины) фильмы категории `isAdult` (выпущенные с 1975 года) по шкале их рейтинга. Посчитать количество кинокартин, попавших в каждый диапазон, для анализа пользовательских оценок в этом сегменте.

```sql
SELECT 
    CASE 
        WHEN "averageRating" < 2.5 THEN '0–2.5'
        WHEN "averageRating" < 4   THEN '2.5–4'
        WHEN "averageRating" < 5.5 THEN '4–5.5'
        WHEN "averageRating" < 7   THEN '5.5–7'
        WHEN "averageRating" < 8.5 THEN '7–8.5'
        ELSE '8.5+' 
    END AS rating_bin,
    COUNT(*) AS cnt_title
FROM imdb.title_basics a
JOIN imdb.title_ratings b ON a.tconst = b.tconst
WHERE "isAdult" = TRUE
  AND "averageRating" IS NOT NULL
  AND "startYear" >= 1975
GROUP BY rating_bin
ORDER BY rating_bin;
```

---

### 📑 Задание 4. Аудит базы данных: поиск рейтингов без метаданных (Записи-сироты)
**Бизнес-задача:** Найти среднее количество голосов для оценок с рейтингом $\ge 7$, для которых в основной таблице `title_basics` отсутствует описание фильма (`a.tconst IS NULL`). Запрос используется для поиска нарушений ссылочной целостности данных.

```sql
SELECT 
    AVG("numVotes") AS avg_numvotes
FROM imdb.title_basics a
RIGHT JOIN imdb.title_ratings b ON a.tconst = b.tconst
WHERE "averageRating" >= 7
  AND a.tconst IS NULL;
```
