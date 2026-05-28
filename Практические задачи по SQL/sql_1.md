# 📊 Решение практических задач по SQL (База данных SkyBank)


В данном проекте представлены решения 4 практических задач по анализу клиентской базы банка `skybank` (таблица `late_collection_clients`). Все запросы оптимизированы, содержат фильтрацию данных, сортировку и ограничение выборки.

---

### 📑 Задание 1. Выборка молодых клиентов с кредитными картами без контактов
**Бизнес-задача:** Найти топ-25 клиентов младше 25 лет с типом кредита 'CC' (Credit Card), у которых не указан номер мобильного телефона, отсортированных по убыванию суммы долга.

```sql
SELECT 
    age, 
    id_client, 
    amt_loan
FROM skybank.late_collection_clients
WHERE age < 25 
  AND credit_type = 'CC' 
  AND cellphone IS NULL
ORDER BY amt_loan DESC
LIMIT 25;
```

---

### 📑 Задание 2. Поиск клиентов с пропусками в профиле (POS-кредитование)
**Бизнес-задача:** Выгрузить демографические данные по POS-кредитам, где в критически важных полях (пол, семейное положение или телефон) присутствуют пустые значения (`NULL`).

```sql
SELECT 
    gender, 
    cellphone, 
    married, 
    credit_type
FROM skybank.late_collection_clients
WHERE credit_type LIKE '%POS%' 
  AND (gender IS NULL OR married IS NULL OR cellphone IS NULL);
```

---

### 📑 Задание 3. Анализ активных возрастных клиентов (Программа TOP-UP)
**Бизнес-задача:** Найти 25 активных клиентов старше 55 лет с доступным номером телефона по продукту 'TOP-UP' и суммой займа менее 100 000. Сортировка по возрастанию суммы кредита.

```sql
SELECT 
    id_client
FROM skybank.late_collection_clients
WHERE credit_type = 'TOP-UP' 
  AND age > 55 
  AND amt_loan < 100000 
  AND is_active = '1'
  AND cellphone IS NOT NULL
ORDER BY amt_loan ASC
LIMIT 25;
```

---

### 📑 Задание 4. Анализ крупных заемщиков по стандартным кредитным продуктам
**Бизнес-задача:** Вывести топ-50 записей по крупным займам (от 250 000 до 500 000), исключая программы 'TOP-UP' и 'POS', отсортированных от самых больших сумм к меньшим.

```sql
SELECT *
FROM skybank.late_collection_clients
WHERE amt_loan BETWEEN 250000 AND 500000
  AND credit_type NOT LIKE 'TOP-UP'
  AND credit_type NOT LIKE '%POS%'
ORDER BY amt_loan DESC
LIMIT 50;
```
