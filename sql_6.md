# 📊 Комплексный анализ и вертикальная интеграция данных (UNION ALL)

В данном разделе собраны сложные аналитические запросы, использующие оператор `UNION ALL` для консолидации метрик из разных временных периодов, продуктовых сегментов и независимых региональных баз данных.

---

### 📑 Задание 1. Расчет Retention Rate режиссеров (Кинематограф IMDb)
**Бизнес-задача:** Оценить долю режиссеров, которые продолжили выпускать картины в следующем году (метрика удержания / Retention). Запрос рассчитывает показатель последовательно для периодов 2018–2019 и 2019–2020 гг. с приведением к типу `float` для точности процентов.

```sql
SELECT 
    '2018-2019' AS period,
    COUNT(DISTINCT b.directors) / COUNT(DISTINCT a.directors)::FLOAT * 100 AS share_directors
FROM imdb.directors_2018 a
LEFT JOIN imdb.directors_2019 b ON a.directors = b.directors
     
UNION ALL

SELECT 
    '2019-2020' AS period,
    COUNT(DISTINCT d.directors) / COUNT(DISTINCT c.directors)::FLOAT * 100 AS share_directors
FROM imdb.directors_2019 c
LEFT JOIN imdb.directors_2020 d ON c.directors = d.directors;
```

---

### 📑 Задание 2. Расчет банковских резервов по просроченным кредитам (Москва)
**Бизнес-задача:** Консолидировать данные по заемщикам из Москвы для расчета финансовых резервов банка. Для портфеля позднего сбора (`late_collection`) резервируется 90% от суммы займа, для раннего сбора (`early_collection`) — 60%.

```sql
SELECT 
    id_client,
    amt_loan,
    amt_loan * 0.9 AS reserv 
FROM skybank.late_collection_clients lcc
JOIN skybank.region_dict rd ON lcc.id_city = rd.id_city
WHERE name_city = 'Москва'
  AND amt_loan IS NOT NULL

UNION ALL

SELECT 
    id_client,
    amt_credit,
    amt_credit * 0.6 AS reserv 
FROM skybank.early_collection_clients ecc
JOIN skybank.region_dict r ON ecc.id_city = r.id_city
WHERE name_city = 'Москва'
  AND amt_credit IS NOT NULL;
```

---

### 📑 Задание 3. Сравнительный анализ стоимости долгосрочной аренды AirBnB
**Бизнес-задача:** Провести бенчмаркинг средней стоимости аренды приватных комнат (`Private room`) на минимальный срок от 30 ночей в трех разных городах США (Даллас, Нью-Йорк, Окленд) на основе изолированных таблиц.

```sql
SELECT 
    'Dallas' AS city,
    AVG(price) AS avg_price
FROM airbnb_dallas.listings
WHERE room_type = 'Private room'
  AND minimum_nights = 30
  
UNION ALL

SELECT 
    'New York' AS city,
    AVG(price) AS avg_price
FROM airbnb_new_york.listings
WHERE room_type = 'Private room'
  AND minimum_nights = 30
  
UNION ALL

SELECT 
    'Oakland' AS city,
    AVG(price) AS avg_price
FROM airbnb_oakland.listings
WHERE room_type = 'Private room'
  AND minimum_nights = 30;
```
