поставить PostgreSQL

*pg_lsclusters*

 ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/71aeacb2-7ab3-4566-8a69-0e5c1f337e80)

------------------------------------------------
запустить везде psql из под пользователя postgres

*sudo -u postgres psql*

 ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/17159052-2a96-4659-a12e-12d32004cccc)
```sql
-----------------------------------------------
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 test      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | 
```
-----------------------------------------------
выключить auto commit

*\set AUTOCOMMIT OFF*

 ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/555de060-d867-465e-906b-5cf06c428f6c)

----------------------------------------------
в первой сессии создать новую таблицу и наполнить ее данными

```sql
create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name)
values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov');
```
```sql
test=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)

```
-----------------------------------------------
посмотреть текущий уровень изоляции: show transaction isolation level

```sql
test=# SHOW default_transaction_isolation;

default_transaction_isolation 
-------------------------------
 read committed
(1 row)

```
------------------------------
в первой сессии добавить новую запись 
```sql
insert into persons(first_name, second_name) values('sergey', 'sergeev');
```
сделать select * from persons во второй сессии

видите ли вы новую запись и если да то почему?

> * Новую запись не вижу.
-------------------------------
завершить первую транзакцию - commit;

сделать select * from persons  во второй сессии

видите ли вы новую запись и если да то почему?
> * Новую запись вижу, так ка первая сессия завершила выполнение своей транзакции, и вторая сессия сможет увидеть новые записи в таблице.
> Уровень изоляции READ COMMITTED(Чтение зафиксированных данных) не позволяет читать грязные данные и пока первая сессия не закончит выполнять свою транзакцию ,т.е. добавлять новую запись в таблицу, вторая сессия не увидит новую запись в таблице, так первая сессия может откатиться.

```sql
test=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
------------------------------
начать новые транзакции но уже repeatable read транзации 
```sql
 set transaction isolation level repeatable read;
```
в первой сессии добавить новую запись
```sql
insert into persons(first_name, second_name) values('sveta', 'svetova');
```
сделать select* from persons во второй сессии
```sql
test=*# select* from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```

видите ли вы новую запись и если да то почему?
> * Новую запись не вижу.

завершить первую транзакцию - commit;

сделать select * from persons во второй сессии

```sql
test=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
видите ли вы новую запись и если да то почему?

> * Новую запись не вижу.

завершить вторую транзакцию

сделать select * from persons во второй сессии

видите ли вы новую запись и если да то почему?

```sql
test=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 rows)
```
> * Новую запись вижу, так ка вторая сессия завершила выполнение своей транзакции и теперь сессия сможет увидеть новые записи в таблице.
> Уровень изоляции REPEATABLE READ 
