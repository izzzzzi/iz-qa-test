# Блок 3: Анализ данных (SQL)

## Задача 1: Вторая зарплата

```sql
-- Дано: Employee(id int PK, salary int)
-- Найти вторую по величине уникальную зарплату, или NULL если её нет

SELECT MAX(salary) AS second_highest_salary
FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);
```

**Пояснение:**
- Внутренний подзапрос находит максимальную зарплату
- Внешний запрос берёт максимум среди всех зарплат, которые меньше этого максимума
- Если уникальных зарплат меньше двух — `MAX(salary)` вернёт `NULL`

**Альтернатива (LIMIT + OFFSET):**

```sql
-- Для PostgreSQL
SELECT salary AS second_highest_salary
FROM Employee
GROUP BY salary
ORDER BY salary DESC
OFFSET 1 LIMIT 1;
```

`OFFSET 1 LIMIT 1` — пропускаем первую (максимальную) и берём следующую. Если её нет — результат пустой, что в PostgreSQL даёт 0 строк вместо NULL. В контексте задачи первый вариант предпочтительнее, так как явно возвращает NULL по условию.

---

## Задача 2: Поиск дубликатов email

```sql
-- Дано: Person(id int PK, email varchar)
-- Найти все дублирующиеся email-адреса

SELECT email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

**Пояснение:**
- Группируем по email
- `HAVING COUNT(*) > 1` оставляет только те группы, где больше одной записи
- Результат — список email, которые встречаются >= 2 раз

**Что ещё можно показать (если нужно больше деталей):**

```sql
-- С количеством дубликатов
SELECT email, COUNT(*) AS dup_count
FROM Person
GROUP BY email
HAVING COUNT(*) > 1
ORDER BY dup_count DESC;
```

---

## Задача 3: Клиенты без заказов

```sql
-- Дано: Customers(id int PK, name varchar)
--        Orders(id int PK, customerId int FK)
-- Найти всех клиентов, которые ни разу не делали заказов

SELECT c.id, c.name
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customerId
WHERE o.id IS NULL;
```

**Пояснение:**
- `LEFT JOIN` — берём всех клиентов, даже если у них нет заказов
- `WHERE o.id IS NULL` — оставляем только тех, у кого в таблице Orders нет совпадений
- Результат — клиенты, у которых `customerId` не встречается в Orders

**Альтернатива через NOT EXISTS:**

```sql
SELECT c.id, c.name
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o WHERE o.customerId = c.id
);
```

`NOT EXISTS` часто эффективнее на больших таблицах — прекращает сканирование при первом совпадении.

---

### Проверка LeetCode

Эти задачи соответствуют уровню Easy/Medium на LeetCode:
- Задача 1 — аналог [176. Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)
- Задача 2 — аналог [182. Duplicate Emails](https://leetcode.com/problems/duplicate-emails/)
- Задача 3 — аналог [183. Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/)

Диалект: PostgreSQL.
