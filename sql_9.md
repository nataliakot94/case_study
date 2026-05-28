# 🎬 Аналитика онлайн-кинотеатра и оконные функции (SkyCinema)


В данном разделе представлены 3 сложных маркетинговых запроса к таблицам `skycinema.client_sign_up` и `skycinema.partner_dict`. Решения построены на базе оконных функций (`ROW_NUMBER`, `LAG`), применяются для анализа продуктовых метрик, исследования поведения когорт и расчета скорости возврата клиентов за повторными покупками (Time to Next Purchase).

---

### 📑 Задание 1. Расчет сквозной и внутрипользовательской хронологии транзакций
**Бизнес-задача:** Обогатить данные по подпискам информацией о партнерах и построить две независимые шкалы времени: `number_purchase` (сквозной порядковый номер покупки по всей платформе) и `purchase_user` (номер покупки индивидуально для каждого пользователя).

```sql
SELECT 
    csu.*, 
    pd.*,
    ROW_NUMBER() OVER(ORDER BY date_purchase) AS number_purchase,
    ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY date_purchase) AS purchase_user
FROM skycinema.client_sign_up AS csu
JOIN skycinema.partner_dict AS pd ON csu.partner = pd.id_partner;
```

---

### 📑 Задание 2. Анализ лидогенерации и первых покупок по партнерам
**Бизнес-задача:** Определить, какие партнерские каналы и типы подписок (`is_trial`) привлекают наибольшее количество абсолютно новых клиентов. Для этого отфильтровать только самые первые транзакции пользователей (`purchase_user = 1`) и посчитать их объем.

* **Вывод / Инсайт:** Наибольший поток новых клиентов зафиксирован по каналу **«Органическая покупка» с бесплатным (пробным) типом подписки** (1135 первых покупок). Это говорит о высокой эффективности естественного трафика и базового онбординга.

```sql
SELECT 
    is_trial,
    name_partner,
    COUNT(*) AS cnt
FROM (
    SELECT 
        csu.*,
        pd.*,
        ROW_NUMBER() OVER(ORDER BY date_purchase ASC) AS number_purchase,
        ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY date_purchase ASC) AS purchase_user
    FROM skycinema.client_sign_up AS csu
    JOIN skycinema.partner_dict AS pd ON csu.partner = pd.id_partner
) AS t
WHERE purchase_user = 1
GROUP BY is_trial, name_partner
ORDER BY cnt DESC;
```

---

### 📑 Задание 3. Расчет скорости совершения повторной покупки (Time to Second Purchase)
**Бизнес-задача:** Рассчитать средний временной интервал (в днях) между первой и второй покупками клиента в разрезе маркетинговых партнеров. С помощью функции `LAG` извлечь дату предыдущей транзакции и отфильтровать пользователей, совершивших строго второй заказ (`purchase_user = 2`).

* **Вывод / Инсайт:** Быстрее всего за второй покупкой возвращаются клиенты, пришедшие через партнера **Тинькофф**. Этот канал обладает максимальной скоростью удержания и вовлечения аудитории.

```sql
SELECT 
    name_partner,
    AVG(date_purchase - lag_rn) AS avg_rn
FROM (
    SELECT 
        user_id,
        date_purchase,
        is_trial,
        name_partner,
        ROW_NUMBER() OVER(ORDER BY date_purchase) AS number_purchase,
        ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY date_purchase) AS purchase_user,
        LAG(csu.date_purchase) OVER(PARTITION BY csu.user_id ORDER BY csu.date_purchase) AS lag_rn
    FROM skycinema.client_sign_up AS csu
    JOIN skycinema.partner_dict AS pd ON csu.partner = pd.id_partner
) AS t
WHERE purchase_user = 2
GROUP BY name_partner
ORDER BY avg_rn;
```
