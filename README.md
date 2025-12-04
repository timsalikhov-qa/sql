Пример базы данных

**Таблица Users:**
```sql
CREATE TABLE Users (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    username VARCHAR(50),
    created_at DATETIME,
    is_active BOOLEAN,
    balance DECIMAL(10, 2)
);
```

Таблица Orders:

```sql
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    amount DECIMAL(10, 2),
    status VARCHAR(20),
    created_at DATETIME,
    FOREIGN KEY (user_id) REFERENCES Users(id)
);
```
Базовые запросы для проверки

1. Поиск дубликатов email

```sql
SELECT email, COUNT(*) as duplicate_count
FROM Users
GROUP BY email
HAVING COUNT(*) > 1;
```

2. Проверка целостности данных после теста

```sql
- Проверка, что у всех заказов есть существующий пользователь
SELECT o.*
FROM Orders o
LEFT JOIN Users u ON o.user_id = u.id
WHERE u.id IS NULL;
```

3. Поиск неактивных пользователей с активными заказами

```sql
SELECT u.id, u.email, u.is_active, COUNT(o.order_id) as active_orders
FROM Users u
JOIN Orders o ON u.id = o.user_id
WHERE u.is_active = 0
  AND o.status IN ('processing', 'shipped')
GROUP BY u.id;
```

4. Проверка бизнес-правил (минимальный баланс)

```sql
- Найти пользователей с отрицательным балансом (не должно быть)
SELECT id, email, balance
FROM Users
WHERE balance < 0;
```

5. Анализ данных для отчета

```sql
- Статистика заказов по дням
SELECT 
    DATE(created_at) as order_date,
    COUNT(*) as total_orders,
    SUM(amount) as total_amount,
    AVG(amount) as avg_order_value
FROM Orders
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY DATE(created_at)
ORDER BY order_date DESC;
```

 Поиск проблемных данных

1. Некорректные email адреса

```sql
SELECT id, email
FROM Users
WHERE email NOT LIKE '%@%.%'
   OR email LIKE '% %'
   OR email LIKE '%@%@%';
```

2. Заказы без пользователей (orphaned records)

```sql
SELECT o.*
FROM Orders o
WHERE NOT EXISTS (
    SELECT 1 FROM Users u WHERE u.id = o.user_id
);
```

3. Проверка валидации статусов

```sql
- Найти заказы с невалидными статусами
SELECT DISTINCT status
FROM Orders
WHERE status NOT IN ('new', 'processing', 'shipped', 'delivered', 'cancelled');
```

4. Поиск тестовых данных

```sql
- Найти тестовых пользователей
SELECT *
FROM Users
WHERE email LIKE '%test%' 
   OR email LIKE '%example%'
   OR username LIKE '%test%';
```

SQL для подготовки тестовых данных

1. Создание тестового пользователя

```sql
INSERT INTO Users (id, email, username, created_at, is_active, balance)
VALUES (999999, 'test.user@example.com', 'testuser', NOW(), 1, 1000.00);
```

2. Очистка тестовых данных после теста

```sql
DELETE FROM Orders WHERE user_id = 999999;
DELETE FROM Users WHERE id = 999999;
```

3. Массовое обновление для тестирования

```sql
- Имитация индексации
UPDATE Users 
SET balance = balance * 1.1 
WHERE created_at < '2025-04-12';
```

Запросы в ежедневной работы

1. Быстрая проверка количества записей

```sql
SELECT 
    (SELECT COUNT(*) FROM Users) as total_users,
    (SELECT COUNT(*) FROM Orders) as total_orders,
    (SELECT COUNT(*) FROM Users WHERE is_active = 1) as active_users;
```

2. Поиск последних действий пользователя

```sql
SELECT u.email, o.status, o.created_at
FROM Users u
JOIN Orders o ON u.id = o.user_id
WHERE u.email = 'customer@example.com'
ORDER BY o.created_at DESC
LIMIT 10;
```

3. Проверка производительности

```sql
- EXPLAIN покажет план выполнения
EXPLAIN SELECT *
FROM Users u
JOIN Orders o ON u.id = o.user_id
WHERE o.amount > 1000
  AND o.created_at > '2024-01-01';
```
