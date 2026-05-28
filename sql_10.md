# 📈 Продвинутый анализ временных рядов и многомерное агрегирование (Advanced Window Functions)

В данном разделе представлены решения сложных аналитических задач финансового и продуктового контуров. Запросы демонстрируют глубокое понимание механики работы оконных функций: расчет скользящих средних (`Moving Average`) с ограничением рамок строк (`ROWS BETWEEN`), а также построение многоуровневой структуры распределения долей по нескольким аналитическим разрезам одновременно.

---

### 📑 Задание 1. Сглаживание временных рядов и расчет скользящих средних (Moving Average)
**Бизнес-задача:** Проанализировать помесячную динамику выручки онлайн-кинотеатра `skycinema`. Для очистки данных от краткосрочных колебаний и выявления глобального тренда реализовать три алгоритма сглаживания: 3-месячное скользящее среднее (`ma_3`), 7-месячное скользящее среднее (`ma_7`) и центрированное (двустороннее) скользящее среднее за 5 месяцев (`ma_2side`).

```sql
SELECT 
    t.*,
    AVG(payment) OVER (ORDER BY mm ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS ma_3,
    AVG(payment) OVER (ORDER BY mm ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS ma_7,
    AVG(payment) OVER (ORDER BY mm ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING) AS ma_2side
FROM (
    SELECT 
        DATE_TRUNC('month', date_purchase) AS mm,
        SUM(amt_payment) AS payment
    FROM skycinema.client_sign_up
    GROUP BY mm 
) AS t
ORDER BY mm ASC;
```

---

### 📑 Задание 2. Структурный анализ кредитного портфеля и риск-метрики (SkyBank)
**Бизнес-задача:** Провести комплексный аудит просроченной задолженности банка. Запрос обогащает профили клиентов числовыми флагами (для дальнейшего скоринга) и параллельно рассчитывает долю каждого займа в трех разрезах: внутри конкретного города (`share_credit_city`), внутри кредитного продукта (`share_credit_type`) и на пересечении города и продукта (`share_credit_type_city`), а также собирает объемы и количество договоров для каждого уровня.

```sql
SELECT 
    id_client,
    name_city,
    CASE WHEN gender = 'M' THEN 1 ELSE 0 END AS nflag_gender,
    age,
    first_time,
    CASE WHEN cellphone IS NOT NULL THEN 1 ELSE 0 END AS nflag_cellphone,
    is_active,
    cl_segm,
    amt_loan,
    date_loan::DATE,
    credit_type,
    
    -- Агрегаты по городам
    SUM(amt_loan) OVER(PARTITION BY name_city) AS sum_city,
    amt_loan::FLOAT / SUM(amt_loan) OVER (PARTITION BY name_city) AS share_credit_city,
    COUNT(*) OVER (PARTITION BY name_city) AS count_loan_city,
    
    -- Агрегаты по типам кредитов
    SUM(amt_loan) OVER (PARTITION BY credit_type) AS sum_type,
    amt_loan::FLOAT / SUM(amt_loan) OVER (PARTITION BY credit_type) AS share_credit_type,
    COUNT(*) OVER (PARTITION BY credit_type) AS count_loan_type,
    
    -- Перекрестные агрегаты (Тип кредита + Город)
    SUM(amt_loan) OVER (PARTITION BY credit_type, name_city) AS sum_loan_type_city,
    amt_loan::FLOAT / SUM(amt_loan) OVER (PARTITION BY credit_type, name_city) AS share_credit_type_city,
    COUNT(*) OVER (PARTITION BY name_city, credit_type) AS count_loan_city_type
FROM skybank.late_collection_clients AS lcc
LEFT JOIN skybank.region_dict AS rd ON lcc.id_city = rd.id_city;
```
