поставить PostgreSQL

*pg_lsclusters*

> ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/71aeacb2-7ab3-4566-8a69-0e5c1f337e80)

------------------------------------------------
запустить везде psql из под пользователя postgres

*sudo -u postgres psql*

> ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/17159052-2a96-4659-a12e-12d32004cccc)
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

> ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/555de060-d867-465e-906b-5cf06c428f6c)

----------------------------------------------
в первой сессии новую таблицу и наполнить ее данными
create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

```sql
1. create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name)
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
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
```sql
insert into persons(first_name, second_name) values('sergey', 'sergeev');
```
сделать select from persons во второй сессии

```sql
select from persons;
```
видите ли вы новую запись и если да то почему?

*новую запись я не вижу, в данный момент уровень изоляции read commited и пока первая сессия  не закончит выполнять свою транзакцию, вторая сессия не увидит новую запись*



