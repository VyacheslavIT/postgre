
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

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/f3f39959-4b61-49db-a40e-739e648a1169)

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/a6e4f38d-9aad-49b8-b8e8-b4fa43b8ed2f)

В статистике по таблицам есть значение количество последовательных чтений, произведённых в этой таблице,количество «живых» строк, прочитанных при последовательных чтениях
количество сканирований по индексу, произведённых в этой таблице = 0

```sql
"Hash Join  (cost=22.15..55.54 rows=1850 width=130)"
"  Hash Cond: (o.user_id = u.user_id)"
"  ->  Seq Scan on orders o  (cost=0.00..28.50 rows=1850 width=16)"
"  ->  Hash  (cost=15.40..15.40 rows=540 width=122)"
"        ->  Seq Scan on users u  (cost=0.00..15.40 rows=540 width=122)"
```
В плане запроса присутствует оператор Hash Join, также присутствует Seq Scan в таблице users  и Seq Scan в таблице orders. Seq Scan является не оптимальным для больших таблиц.
Следует отметить, что полный просмотр таблицы далеко не всегда медленнее просмотра по индексу. Если, например, в таблице–справочнике 
несколько сотен записей, умещающихся в одном-двух блоках на диске, то использование индекса приведёт лишь к тому, что придётся читать ещё
и пару лишних блоков индекса. Если в запросе придётся выбрать 80% записей из большой таблицы, то полный просмотр опять же получится быстрее.

Обновим статистику по таблицам 

```sql
ANALYZE orders
ANALYZE users

"QUERY PLAN"
"Hash Join  (cost=1.07..2.12 rows=3 width=17)"
"  Hash Cond: (o.user_id = u.user_id)"
"  ->  Seq Scan on orders o  (cost=0.00..1.03 rows=3 width=16)"
"  ->  Hash  (cost=1.03..1.03 rows=3 width=9)"
"        ->  Seq Scan on users u  (cost=0.00..1.03 rows=3 width=9)"

"Hash Join  (cost=1.07..2.12 rows=3 width=17) (actual time=0.069..0.071 rows=3 loops=1)"
"  Hash Cond: (o.user_id = u.user_id)"
"  ->  Seq Scan on orders o  (cost=0.00..1.03 rows=3 width=16) (actual time=0.041..0.041 rows=3 loops=1)"
"  ->  Hash  (cost=1.03..1.03 rows=3 width=9) (actual time=0.020..0.020 rows=3 loops=1)"
"        Buckets: 1024  Batches: 1  Memory Usage: 9kB"
"        ->  Seq Scan on users u  (cost=0.00..1.03 rows=3 width=9) (actual time=0.016..0.017 rows=3 loops=1)"
"Planning Time: 0.131 ms"
"Execution Time: 0.092 ms"

```

Стоимость запроса снизилась с 55.54 до 2.12, кол-во выбранных строк совпадает с кол-вом строк в таблицах. 

Добавляем индексы 

CREATE INDEX ON bookings.orders (user_id);

CREATE INDEX ON bookings.users (user_id);

Отключим seqscan, чтобы планировщик использовал индексы
SET enable_seqscan = OFF;

```
explain ANALYZE
SELECT u.username, o.order_id, o.order_date, o.total_amount
FROM users u
JOIN orders o ON u.user_id = o.user_id;
```


```sql
"QUERY PLAN"
"Nested Loop  (cost=0.26..22.61 rows=3 width=130)"
"  ->  Index Scan using orders_user_id_idx on orders o  (cost=0.13..12.18 rows=3 width=16)"
"  ->  Index Scan using users_user_id_idx on users u  (cost=0.13..4.15 rows=1 width=122)"
"        Index Cond: (user_id = o.user_id)"

"QUERY PLAN"
"Nested Loop  (cost=0.27..20.58 rows=3 width=17) (actual time=0.057..0.061 rows=3 loops=1)"
"  ->  Index Scan using orders_user_id_idx on orders o  (cost=0.13..12.18 rows=3 width=16) (actual time=0.034..0.035 rows=3 loops=1)"
"  ->  Memoize  (cost=0.14..4.16 rows=1 width=9) (actual time=0.008..0.008 rows=1 loops=3)"
"        Cache Key: o.user_id"
"        Cache Mode: logical"
"        Hits: 1  Misses: 2  Evictions: 0  Overflows: 0  Memory Usage: 1kB"
"        ->  Index Scan using users_user_id_idx on users u  (cost=0.13..4.15 rows=1 width=9) (actual time=0.009..0.009 rows=1 loops=2)"
"              Index Cond: (user_id = o.user_id)"
"Planning Time: 0.103 ms"
"Execution Time: 0.078 ms"

```

Поиск идет в индексах, оператор Nested Loop.
Стоимость повысилась с 2.12 до 22.61 время выполнеия запроса с индексами 0.078 ms без индексов 0.092 ms,кол-во выбранных строк совпадает с кол-вом строк в таблицах.

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/e2ce229e-d6d7-4c63-afa8-0411c39f9dc3)

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/d0b9f711-0cf0-4907-88cd-440365d55393)

Появилось значение - количество сканирований по индексу, произведённых в этой таблице и количество «живых» строк, отобранных при сканированиях по индексу 


Сделаем большие таблицы 
```sql
CREATE TABLE table1 (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO table1 (id, name)
SELECT seq, 'Name' || seq
FROM generate_series(1, 1000000) as seq;

CREATE TABLE table2 (
    id INT PRIMARY KEY,
    value INT
);

INSERT INTO table2 (id, value)
SELECT seq, seq * 2
FROM generate_series(1, 1000000) as seq;
```
```sql
CREATE INDEX ON table1 (id);

CREATE INDEX ON table2  (id);

SET enable_seqscan = OFF;

explain ANALYZE SELECT t1.id, t1.name, t2.value
FROM table1 t1
INNER JOIN table2 t2 ON t1.id = t2.id;

```
```sql

"QUERY PLAN"
"Merge Join  (cost=1.48..76796.56 rows=1000000 width=18) (actual time=0.010..300.443 rows=1000000 loops=1)"
"  Merge Cond: (t1.id = t2.id)"
"  ->  Index Scan using table1_id_idx on table1 t1  (cost=0.42..31389.42 rows=1000000 width=14) (actual time=0.005..81.721 rows=1000000 loops=1)"
"  ->  Index Scan using table2_id_idx on table2 t2  (cost=0.42..30408.42 rows=1000000 width=8) (actual time=0.003..77.661 rows=1000000 loops=1)"
"Planning Time: 0.138 ms"
"Execution Time: 315.161 ms"

```
Планировщик решает использовать Merge Join, время выполнение запроса 315.161 стоимость 76796.56,кол-во выбранных строк совпадает с количеством строк в таблице.

```sql
SET enable_seqscan = ON;

"QUERY PLAN"
"Hash Join  (cost=30832.00..62536.01 rows=1000000 width=18) (actual time=157.784..549.551 rows=1000000 loops=1)"
"  Hash Cond: (t1.id = t2.id)"
"  ->  Seq Scan on table1 t1  (cost=0.00..15406.00 rows=1000000 width=14) (actual time=0.008..33.669 rows=1000000 loops=1)"
"  ->  Hash  (cost=14425.00..14425.00 rows=1000000 width=8) (actual time=157.102..157.103 rows=1000000 loops=1)"
"        Buckets: 262144  Batches: 8  Memory Usage: 6935kB"
"        ->  Seq Scan on table2 t2  (cost=0.00..14425.00 rows=1000000 width=8) (actual time=0.007..59.481 rows=1000000 loops=1)"
"Planning Time: 0.194 ms"
"Execution Time: 564.862 ms"
```
 Планировщик решает использовать Hash Join, время выполнение запроса 564.862 ms стоимость 62536.01,кол-во выбранных строк совпадает с количеством строк в таблице.
 
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

```sql

"Hash Right Join  (cost=22.15..38.85 rows=540 width=236)"
"  Hash Cond: (e.department_id = d.department_id)"
"  ->  Seq Scan on employees e  (cost=0.00..15.30 rows=530 width=122)"
"  ->  Hash  (cost=15.40..15.40 rows=540 width=122)"
"        ->  Seq Scan on departments d  (cost=0.00..15.40 rows=540 width=122)"

```

Обновим статистику в таблице и создадим индексы 
```sql
analyze departments
analyze employees

CREATE INDEX ON departments (department_id);
CREATE INDEX ON employees (department_id)

```
```sql
explain analyze
SELECT d.department_name, e.employee_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id;

```


```sql
"QUERY PLAN"
"Hash Right Join  (cost=1.07..2.13 rows=3 width=9) (actual time=0.017..0.021 rows=4 loops=1)"
"  Hash Cond: (e.department_id = d.department_id)"
"  ->  Seq Scan on employees e  (cost=0.00..1.04 rows=4 width=9) (actual time=0.003..0.003 rows=4 loops=1)"
"  ->  Hash  (cost=1.03..1.03 rows=3 width=8) (actual time=0.012..0.012 rows=3 loops=1)"
"        Buckets: 1024  Batches: 1  Memory Usage: 9kB"
"        ->  Seq Scan on departments d  (cost=0.00..1.03 rows=3 width=8) (actual time=0.009..0.009 rows=3 loops=1)"
"Planning Time: 0.093 ms"
"Execution Time: 0.036 ms"

``` 
Планировщик использует оператор Hash Right Join стоимость 2.13,кол-во выбранных строк совпадает с кол-вом строк в таблицах. 

```sql
SET enable_seqscan = off
```

```sql
"QUERY PLAN"
"Merge Left Join  (cost=0.26..24.41 rows=3 width=9) (actual time=0.014..0.017 rows=4 loops=1)"
"  Merge Cond: (d.department_id = e.department_id)"
"  ->  Index Scan using departments_department_id_idx on departments d  (cost=0.13..12.18 rows=3 width=8) (actual time=0.009..0.010 rows=3 loops=1)"
"  ->  Index Scan using employees_department_id_idx on employees e  (cost=0.13..12.19 rows=4 width=9) (actual time=0.002..0.003 rows=4 loops=1)"
"Planning Time: 0.095 ms"
"Execution Time: 0.031 ms"

```
Планировщик использует оператор Merge Left Join стоимость 24.41 время выполнения отличается незначительно,кол-во выбранных строк совпадает с кол-вом строк в таблицах.

Сделаем большие таблицы 

```sql
CREATE TABLE table1 (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO table1 (id, name)
SELECT
    n, CONCAT('Name ', n)
FROM 
    generate_series(1, 1000000) n;


CREATE TABLE table2 (
    id INT PRIMARY KEY,
    value VARCHAR(50)
);

INSERT INTO table2 (id, value)
SELECT
    n, CONCAT('Value ', n)
FROM 
    generate_series(1, 1000000) n;

SELECT t1.id, t1.name, t2.value
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id;

analyze table1
analyze table2

```

```sql
"QUERY PLAN"
"Hash Right Join  (cost=32789.00..66337.01 rows=1000000 width=27) (actual time=117.475..531.383 rows=1000000 loops=1)"
"  Hash Cond: (t2.id = t1.id)"
"  ->  Seq Scan on table2 t2  (cost=0.00..16274.00 rows=1000000 width=16) (actual time=0.023..62.471 rows=1000000 loops=1)"
"  ->  Hash  (cost=15406.00..15406.00 rows=1000000 width=15) (actual time=116.757..116.758 rows=1000000 loops=1)"
"        Buckets: 262144  Batches: 8  Memory Usage: 7898kB"
"        ->  Seq Scan on table1 t1  (cost=0.00..15406.00 rows=1000000 width=15) (actual time=0.008..33.850 rows=1000000 loops=1)"
"Planning Time: 0.205 ms"
"Execution Time: 546.754 ms"

```

```sql
SET enable_seqscan = OFF;
```

```sql

"QUERY PLAN"
"Merge Left Join  (cost=1.34..78646.85 rows=1000000 width=27) (actual time=0.032..331.193 rows=1000000 loops=1)"
"  Merge Cond: (t1.id = t2.id)"
"  ->  Index Scan using table1_id_idx on table1 t1  (cost=0.42..31389.42 rows=1000000 width=15) (actual time=0.006..93.611 rows=1000000 loops=1)"
"  ->  Index Scan using table2_id_idx on table2 t2  (cost=0.42..32257.42 rows=1000000 width=16) (actual time=0.023..94.066 rows=1000000 loops=1)"
"Planning Time: 0.148 ms"
"Execution Time: 346.146 ms"

```

На больших таблицах операторы остались прежними, время выполнения 346ms, стоимость запроса возросла,кол-во выбранных строк совпадает с количеством строк в таблице.

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

```sql
"QUERY PLAN"
"Nested Loop  (cost=0.00..3677.15 rows=291600 width=236)"
"  ->  Seq Scan on table1 t1  (cost=0.00..15.40 rows=540 width=118)"
"  ->  Materialize  (cost=0.00..18.10 rows=540 width=118)"
"        ->  Seq Scan on table2 t2  (cost=0.00..15.40 rows=540 width=118)"

```


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

```sql
"QUERY PLAN"
"Hash Full Join  (cost=22.15..38.85 rows=540 width=236)"
"  Hash Cond: (e.department_id = d.department_id)"
"  ->  Seq Scan on employees e  (cost=0.00..15.30 rows=530 width=122)"
"  ->  Hash  (cost=15.40..15.40 rows=540 width=122)"
"        ->  Seq Scan on departments d  (cost=0.00..15.40 rows=540 width=122)"

```

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

Создаем таблицу address
CREATE TABLE address (
    address_id SERIAL PRIMARY KEY,
    customer_id INT,
    customer_address VARCHAR(50)
);

Создаем таблицу items_order
CREATE TABLE items_order (
    items_id SERIAL PRIMARY KEY,
	customer_id INT,
    items_name VARCHAR(50)
);

Вставляем данные в таблицу orders
INSERT INTO orders (customer_id, order_date) VALUES (1, '2024-01-15'), (2, '2024-01-20'), (1, '2024-02-05');

Вставляем данные в таблицу customers
INSERT INTO customers (customer_name) VALUES ('Alice'), ('Bob');

Вставляем данные в таблицу address
INSERT INTO address (customer_id, customer_address) VALUES (1, 'Moscow'), (2, 'Kazan'), (3, 'Ekaterinburg');

Вставляем данные в таблицу address
INSERT INTO items_order (customer_id, items_name) VALUES (1, 'Шоколад'), (2, 'Машина'), (3, 'Конфеты');

Запрос с разными типами соединений
SELECT o.order_id, c.customer_name, a.customer_address, o.order_date,i.items_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
LEFT JOIN address a ON c.customer_id = a.customer_id
FULL JOIN items_order i ON c.customer_id =i.customer_id

```



![image](https://github.com/VyacheslavIT/postgre/assets/136000255/576a9ca6-0f7a-41ac-b9c3-ac1ea4c300d7)

```sql
"QUERY PLAN"
"Hash Full Join  (cost=129.55..168.85 rows=2040 width=362)"
"  Hash Cond: (i.customer_id = c.customer_id)"
"  ->  Seq Scan on items_order i  (cost=0.00..15.30 rows=530 width=122)"
"  ->  Hash  (cost=104.05..104.05 rows=2040 width=248)"
"        ->  Hash Join  (cost=45.60..104.05 rows=2040 width=248)"
"              Hash Cond: (o.customer_id = c.customer_id)"
"              ->  Seq Scan on orders o  (cost=0.00..30.40 rows=2040 width=12)"
"              ->  Hash  (cost=38.85..38.85 rows=540 width=240)"
"                    ->  Hash Right Join  (cost=22.15..38.85 rows=540 width=240)"
"                          Hash Cond: (a.customer_id = c.customer_id)"
"                          ->  Seq Scan on address a  (cost=0.00..15.30 rows=530 width=122)"
"                          ->  Hash  (cost=15.40..15.40 rows=540 width=122)"
"                                ->  Seq Scan on customers c  (cost=0.00..15.40 rows=540 width=122)"

```


----------------------------------


* Задание со звездочкой*  Придумайте 3 своих метрики на основе показанных представлений, отправьте их через ЛК, а так же поделитесь с коллегами в слаке

```sql

Метрика для количества выполненных запросов по типу операции
CREATE VIEW query_type_count AS
SELECT queryid, query, calls
FROM pg_stat_statements
WHERE query LIKE 'select%' OR query LIKE 'update%' OR query LIKE 'delete%' order by query;


Метрика для 10 самых медленных запросов
CREATE VIEW slowest_queries AS
SELECT queryid,query,(total_exec_time)/60000 as total_time_sec
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;


Метрика содержит информацию о времени выполнения запросов.
CREATE VIEW query_execution_times_view AS
SELECT pid, query_start, now() - query_start AS execution_time
FROM pg_stat_activity
WHERE state = 'active';

Метрика содержит информацию о текущих активных соединениях к базе данных.
CREATE VIEW active_connections_view AS
SELECT pid, application_name, client_addr, query
FROM pg_stat_activity
WHERE state = 'active';

```


----------------------------------
  
