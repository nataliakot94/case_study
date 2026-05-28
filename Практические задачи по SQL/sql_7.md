# 🔬 Проверка продуктовых гипотез и сложные подзапросы (A/B тестирование)

В данном разделе представлены продвинутые аналитические запросы к базе данных `skyeng_db`. Набор задач направлен на проверку бизнес-гипотез, сегментацию «тяжелых» пользователей (VIP-клиентов) через вложенные подзапросы и сведение метрик из разных операционных контуров (финансы и логи) с помощью `FULL JOIN`.

---

### 📑 Задание 1. Проверка гипотезы о длительности уроков VIP-клиентов (Через подзапрос в CASE WHEN)
**Бизнес-гипотеза:** Клиенты, которые совершили более 3 успешных оплат и покупают в среднем крупные пакеты курсов ($\ge 25$ занятий), склонны дольше заниматься на уроках. 
**Реализация:** Выделение сегмента VIP-пользователей через подзапрос внутри конструкции `CASE WHEN` и расчет среднего хронометража их успешных занятий за 2016 год.

```sql
SELECT 
    CASE 
        WHEN user_id IN (
            SELECT user_id
            FROM skyeng_db.payments
            WHERE status_name = 'success'
            GROUP BY user_id
            HAVING COUNT(*) > 3 AND AVG(classes) > 25
        ) THEN 1 
        ELSE 0 
    END AS opt,
    AVG(class_end_datetime - class_start_datetime) AS avg_len                    
FROM skyeng_db.classes
WHERE DATE_PART('year', class_start_datetime) = 2016
  AND class_status = 'success'
  AND (class_end_datetime - class_start_datetime) BETWEEN INTERVAL '20 minute' AND INTERVAL '3 hour'
GROUP BY opt;
```

---

### 📑 Задание 2. Оптимизация проверки гипотезы (Через подзапрос в LEFT JOIN)
**Бизнес-задача:** Решить задачу из Задания 1 альтернативным, более оптимальным способом — через предварительное агрегирование VIP-клиентов во временную таблицу и её последующее левое соединение (`LEFT JOIN`).
**Результат анализа:** Оба запроса (Задание 1 и 2) показали идентичные средние значения длительности уроков для тестовой и контрольной групп.

* **Вывод / Ответ на гипотезу:** **Гипотеза не подтверждается.** Наличие крупных покупок и лояльность клиента не влияют на среднюю длительность одного занятия.

```sql
SELECT 
    CASE WHEN t.user_id IS NOT NULL THEN 1 ELSE 0 END AS opt,
    AVG(class_end_datetime - class_start_datetime) AS avg_len                    
FROM skyeng_db.classes c
LEFT JOIN (
    SELECT user_id
    FROM skyeng_db.payments
    WHERE status_name = 'success'
    GROUP BY user_id
    HAVING COUNT(*) > 3 AND AVG(classes) > 25
) t ON c.user_id = t.user_id
WHERE DATE_PART('year', class_start_datetime) = 2016
  AND class_status = 'success'
  AND (class_end_datetime - class_start_datetime) BETWEEN INTERVAL '20 minute' AND INTERVAL '3 hour'
GROUP BY opt;
```

---

### 📑 Задание 3. Сведение финансовой и операционной отчетности за 2017 год (Синхронизация по FULL JOIN)
**Бизнес-гипотеза:** Существует ли сезонная корреляция между месячным объемом входящих платежей и фактической нагрузкой на преподавателей (количеством уроков и активных студентов)?
**Реализация:** Агрегация финансовых поступлений из таблицы `payments` и логов занятий из таблицы `classes` за 2017 год с последующим полным соединением (`FULL JOIN`) по месячной шкале для исключения потери данных.

* **Вывод / Ответ на гипотезу:** **Гипотеза подтверждается.** Наблюдается прямая корреляция: сезонные пики и спады в оплатах (например, летнее затишье) синхронно отражаются на количестве проводимых уроков и падении DAU/MAU.

```sql
SELECT 
    t1.mm,
    sum_pay,
    cnt_class,
    cnt_user
FROM (
    SELECT 
        EXTRACT('month' FROM transaction_datetime) AS mm,
        SUM(payment_amount) AS sum_pay
    FROM skyeng_db.payments p
    WHERE DATE_PART('year', transaction_datetime) = 2017
      AND status_name = 'success'
    GROUP BY mm
) t1
FULL JOIN (
    SELECT 
        EXTRACT('month' FROM class_start_datetime) AS mm,
        COUNT(DISTINCT user_id) AS cnt_user,
        COUNT(id_class) AS cnt_class
    FROM skyeng_db.classes c
    WHERE DATE_PART('year', class_start_datetime) = 2017
      AND class_status = 'success'
    GROUP BY mm
) t2 ON t1.mm = t2.mm;
```
