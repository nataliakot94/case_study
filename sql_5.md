# 🛒 Аналитика розничных продаж и E-commerce

В данном разделе представлены 3 комплексных аналитических запроса к связанным таблицам `purchases`, `product_list` и `products_price_log`. Задачи сфокусированы на работе со сложной логикой изменения цен во времени, когортном анализе (округление дат) и расчете скользящих временных окон для сегментации клиентов.

---

### 📑 Задание 1. Определение даты запуска продаж продуктовых товаров
**Бизнес-задача:** Для каждого продукта из категории 'Food' определить точную дату его самой первой покупки на платформе (дата вывода товара на рынок).

```sql
SET datestyle TO 'iso, dmy';

SELECT 
    name_product,
    MIN(dtime_purchase::DATE) AS min_date
FROM purchases AS p
JOIN product_list AS pl ON p.id_product = pl.id_product
WHERE type = 'Food'
GROUP BY name_product;
```

---

### 📑 Задание 2. Расчет ежемесячной выручки по категориям с восстановлением исторических цен
**Бизнес-задача:** Рассчитать ежемесячную выручку (`revenue`) в разрезе продуктовых категорий. Так как цена товара меняется, стоимость каждой покупки динамически восстанавливается из лога цен (`products_price_log`) на момент совершения транзакции.

```sql
SET datestyle TO 'iso, dmy';

SELECT 
    type, 
    DATE_TRUNC('month', dtime_purchase::DATE) AS month_purchase, 
    SUM(price) AS revenue
FROM purchases AS p
JOIN product_list AS pl ON pl.id_product = p.id_product
JOIN products_price_log AS ppl ON p.id_product = ppl.id_product
  AND dtime_purchase::DATE BETWEEN dttm_from::DATE AND dttm_to::DATE
GROUP BY type, month_purchase;
```

---

### 📑 Задание 3. Выявление VIP-клиентов за последние 2 месяца
**Бизнес-задача:** Найти ID клиентов, у которых суммарный чек покупок за последние 2 месяца (относительно отчетной даты 24.08.2024) превысил 50 000 рублей. Запрос используется для формирования сегмента лояльности и запуска таргетированных кампаний.

```sql
SET datestyle TO 'iso, dmy';

SELECT 
    id_client,
    SUM(price) AS revenue
FROM purchases AS p
JOIN product_list AS pl ON pl.id_product = p.id_product
JOIN products_price_log AS ppl ON p.id_product = ppl.id_product
  AND dtime_purchase::DATE BETWEEN dttm_from::DATE AND dttm_to::DATE
WHERE dtime_purchase::DATE >= DATE '2024-08-24' - INTERVAL '2 month'
GROUP BY id_client
HAVING SUM(price) > 50000;
```
