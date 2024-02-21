
* Реализовать прямое соединение двух или более таблиц
```sql
  Создаем таблицу users
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50)
);

 Создаем таблицу orders
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT,
    order_date DATE,
    total_amount INT
);

Вставляем данные в таблицу users
INSERT INTO users (username) VALUES ('Maxim'), ('Dima'), ('Misha');

Вставляем данные в таблицу orders
INSERT INTO orders (user_id, order_date, total_amount) VALUES
(1, '2024-02-19', 100),
(2, '2024-02-20', 75),
(1, '2024-02-21', 50);

Запрос с прямым соединением таблиц users и orders
SELECT u.username, o.order_id, o.order_date, o.total_amount
FROM users u
JOIN orders o ON u.user_id = o.user_id;
```


Результатом запроса будет таблица, содержащая данные из обеих таблиц, объединенных по условию равенства значений столбца "user_id".

--------------------------------



* Реализовать левостороннее (или правостороннее) соединение двух или более таблиц
 ```sql 
Создаем таблицу departments
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(50)
);

Создаем таблицу employees
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    employee_name VARCHAR(50),
    department_id INT
);

Вставляем данные в таблицу departments
INSERT INTO departments (department_name) VALUES ('HR'), ('IT'), ('Finance');

Вставляем данные в таблицу employees
INSERT INTO employees (department_id, employee_name) VALUES
(1,'Maxim'),
(2,'Dima'),
(1,'Misha'),
(NULL,'Gena');

Запрос с левосторонним соединением таблиц departments и employees
SELECT d.department_name, e.employee_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id;
```
Результатом запроса будет таблица, содержащая все отделы из таблицы "departments" и соответствующих сотрудников из таблицы "employees". 
Если для отдела нет соответствующих сотрудников, то в результирующей таблице будут значения NULL для столбца сотрудников.

---------------------------------


* Реализовать кросс соединение двух или более таблиц

```sql

Создаем таблицу table1
CREATE TABLE table1 (
    id SERIAL PRIMARY KEY,
    column1 VARCHAR(50)
);

Создаем таблицу table2
CREATE TABLE table2 (
    id SERIAL PRIMARY KEY,
    column2 VARCHAR(50)
);

Вставляем данные в таблицу table1
INSERT INTO table1 (column1) VALUES ('A'), ('B'), ('C');

Вставляем данные в таблицу table2
INSERT INTO table2 (column2) VALUES ('X'), ('Y'), ('Z');

Запрос с кросс-соединением таблиц table1 и table2
SELECT t1.column1, t2.column2
FROM table1 t1
CROSS JOIN table2 t2;


```

Результатом запроса будет таблица, содержащая все возможные комбинации строк из обеих таблиц.

---------------------------------


* Реализовать полное соединение двух или более таблиц


----------------------------------

* Реализовать запрос, в котором будут использованы разные типы соединений

----------------------------------

Сделать комментарии на каждый запрос К работе приложить структуру таблиц, для которых выполнялись соединения

----------------------------------


* Задание со звездочкой*  Придумайте 3 своих метрики на основе показанных представлений, отправьте их через ЛК, а так же поделитесь с коллегами в слаке

----------------------------------
  
