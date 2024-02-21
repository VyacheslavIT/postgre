
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

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/1638fc80-4c6d-4056-82dd-dd65e625dde3)




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

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/b7917c71-f4f3-415b-b42e-e1404334a81a)

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

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/727bb94f-0293-4c8a-b6cc-b749dd457685)


---------------------------------


* Реализовать полное соединение двух или более таблиц

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
INSERT INTO departments (department_name) VALUES ('HR'), ('IT'), ('Finance'),('Marketing');

Вставляем данные в таблицу employees
INSERT INTO employees (department_id, employee_name) VALUES
(1,'Maxim'),
(2,'Dima'),
(1,'Misha'),
(NULL,'Gena'),
(null,'Nadya'),
(null,'Lena'),
(null,'Marina');

Запрос с левосторонним соединением таблиц departments и employees
SELECT d.department_name, e.employee_name
FROM departments d
FULL JOIN employees e ON d.department_id = e.department_id;

```
Результатом запроса будет таблица, содержащая все строки из обеих таблиц, дополняя отсутствующие значения NULL.

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/cabf2d72-e0cf-4407-8302-d191d1a51cfb)


----------------------------------

* Реализовать запрос, в котором будут использованы разные типы соединений
  
```sql
Создаем таблицу orders
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE
);

Создаем таблицу customers
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(50)
);

Вставляем данные в таблицу orders
INSERT INTO orders (customer_id, order_date) VALUES (1, '2024-01-15'), (2, '2024-01-20'), (1, '2024-02-05');

Вставляем данные в таблицу customers
INSERT INTO customers (customer_name) VALUES ('Alice'), ('Bob');

Запрос с разными типами соединений
SELECT o.order_id, c.customer_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2024-01-01'
UNION
SELECT o.order_id, c.customer_name
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL
UNION
SELECT o.order_id, c.customer_name
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date < '2024-02-01';

```

1 INNER JOIN: Возвращает только те строки, для которых есть соответствие в обеих таблицах.
2 LEFT JOIN: Возвращает все строки из левой таблицы (orders) и соответствующие строки из правой таблицы (customers), если они есть.
3 RIGHT JOIN: Возвращает все строки из правой таблицы (customers) и соответствующие строки из левой таблицы (orders), если они есть.
Запрос объединяет данные из таблиц "orders" и "customers" с использованием различных типов соединений и добавляет условия для фильтрации результатов.
----------------------------------

Сделать комментарии на каждый запрос К работе приложить структуру таблиц, для которых выполнялись соединения

----------------------------------


* Задание со звездочкой*  Придумайте 3 своих метрики на основе показанных представлений, отправьте их через ЛК, а так же поделитесь с коллегами в слаке

----------------------------------
  
