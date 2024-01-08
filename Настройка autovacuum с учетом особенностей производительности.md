
* Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
* Установить на него PostgreSQL 15 с дефолтными настройками
  
![image](https://github.com/VyacheslavIT/postgre/assets/136000255/7b1aebe5-1948-45aa-82c1-ff49fcf23e47)


 ------------------- 
* Создать БД для тестов: выполнить pgbench -i postgres
```sql
test_outus=# create database pgbench;

CREATE DATABASE
```


postgres@slavavm1:/home/slava$ pgbench -i pgbench;

dropping old tables...

NOTICE:  table "pgbench_accounts" does not exist, skipping

NOTICE:  table "pgbench_branches" does not exist, skipping

NOTICE:  table "pgbench_history" does not exist, skipping

NOTICE:  table "pgbench_tellers" does not exist, skipping

creating tables...
generating data (client-side)...

100000 of 100000 tuples (100%) done (elapsed 0.04 s, remaining 0.00 s)

vacuuming...

creating primary keys...

done in 0.21 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 0.12 s, vacuum 0.03 s, primary keys 0.05 s).

postgres@slavavm1:/home/slava$ 
```sql
pgbench=# \dt
              List of relations
 Schema |       Name       | Type  |  Owner   
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
(4 rows)

```
 ------------------ 
* Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
  
postgres@slavavm1:/home/slava$ pgbench -c8 -P 6 -T 60 -U postgres pgbench

pgbench (15.5 (Ubuntu 15.5-1.pgdg23.10+1))

starting vacuum...end.

progress: 6.0 s, 890.8 tps, lat 8.960 ms stddev 5.295, 0 failed

progress: 12.0 s, 835.8 tps, lat 9.566 ms stddev 5.556, 0 failed

progress: 18.0 s, 810.6 tps, lat 9.869 ms stddev 5.908, 0 failed

progress: 24.0 s, 784.5 tps, lat 10.085 ms stddev 5.692, 0 failed

progress: 30.0 s, 774.7 tps, lat 10.434 ms stddev 28.979, 0 failed

progress: 36.0 s, 866.0 tps, lat 9.245 ms stddev 5.111, 0 failed

progress: 42.0 s, 845.8 tps, lat 9.456 ms stddev 5.372, 0 failed

progress: 48.0 s, 838.8 tps, lat 9.531 ms stddev 5.390, 0 failed

progress: 54.0 s, 813.8 tps, lat 9.836 ms stddev 5.735, 0 failed

progress: 60.0 s, 792.1 tps, lat 10.098 ms stddev 5.821, 0 failed

transaction type: <builtin: TPC-B (sort of)>

scaling factor: 1

query mode: simple

number of clients: 8

number of threads: 1

maximum number of tries: 1

duration: 60 s

number of transactions actually processed: 49526

number of failed transactions: 0 (0.000%)

latency average = 9.691 ms

latency stddev = 10.334 ms

initial connection time = 9.302 ms

tps = 825.433210 (without initial connection time)

postgres@slavavm1:/home/slava$ 

  
-------------------  
* Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
```sql
postgres=# SHOW max_connections;
 max_connections 
-----------------
 40
(1 row)

postgres=# SHOW shared_buffers;
 shared_buffers 
----------------
 1GB
(1 row)

postgres=# SHOW effective_cache_size;
 effective_cache_size 
----------------------
 3GB
(1 row)

postgres=# SHOW maintenance_work_mem;
 maintenance_work_mem 
----------------------
 512MB
(1 row)

postgres=# SHOW checkpoint_completion_target;
 checkpoint_completion_target 
------------------------------
 0.9
(1 row)

postgres=# SHOW wal_buffers;
 wal_buffers 
-------------
 16MB
(1 row)

postgres=# SHOW default_statistics_target;
 default_statistics_target 
---------------------------
 500
(1 row)

postgres=# SHOW random_page_cost;
 random_page_cost 
------------------
 4
(1 row)

postgres=# SHOW effective_io_concurrency;
 effective_io_concurrency 
--------------------------
 2
(1 row)

postgres=# SHOW work_mem;
 work_mem 
----------
 6553kB
(1 row)

postgres=# SHOW min_wal_size;
 min_wal_size 
--------------
 4GB
(1 row)

postgres=# SHOW max_wal_size;
 max_wal_size 
--------------
 16GB
(1 row)
```
--------------------------
* Протестировать заново

postgres@slavavm1:/home/slava$ pgbench -c8 -P 6 -T 60 -U postgres pgbench

pgbench (15.5 (Ubuntu 15.5-1.pgdg23.10+1))

starting vacuum...end.

progress: 6.0 s, 703.8 tps, lat 11.334 ms stddev 7.483, 0 failed

progress: 12.0 s, 821.9 tps, lat 9.732 ms stddev 5.702, 0 failed

progress: 18.0 s, 882.9 tps, lat 9.057 ms stddev 5.061, 0 failed

progress: 24.0 s, 880.5 tps, lat 9.087 ms stddev 5.218, 0 failed

progress: 30.0 s, 891.9 tps, lat 8.969 ms stddev 4.889, 0 failed

progress: 36.0 s, 889.5 tps, lat 8.995 ms stddev 5.178, 0 failed

progress: 42.0 s, 875.8 tps, lat 9.131 ms stddev 5.235, 0 failed

progress: 48.0 s, 844.5 tps, lat 9.477 ms stddev 5.484, 0 failed

progress: 54.0 s, 811.3 tps, lat 9.854 ms stddev 5.450, 0 failed

progress: 60.0 s, 794.3 tps, lat 10.076 ms stddev 5.734, 0 failed

transaction type: <builtin: TPC-B (sort of)>

scaling factor: 1

query mode: simple

number of clients: 8

number of threads: 1

maximum number of tries: 1

duration: 60 s

number of transactions actually processed: 50387

number of failed transactions: 0 (0.000%)

latency average = 9.525 ms

latency stddev = 5.577 ms

initial connection time = 11.557 ms

tps = 839.758560 (without initial connection time)

-------------------------
* Что изменилось и почему?
  
-------------------------
* Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
```sql
create table test (c1 text);

test_outus=# select count(*) from test;
 count 
-------
     0
(1 row)

```
```sql

CREATE FUNCTION insert_test()  
RETURNS void AS
$BODY$
DECLARE
  rec text;
BEGIN
  FOR rec IN 1..1000000 LOOP
    INSERT INTO test VALUES (rec);
  END LOOP;
END;
$BODY$
  LANGUAGE plpgsql;
  
 select insert_test();

test_outus=# select count(*) from test;
  count  
---------
 1000000
(1 row)


```
или можно так 

```sql
INSERT INTO test SELECT  FROM generate_series(1,1000000);
```
 ---------------------- 
* Посмотреть размер файла с таблицей
```sql
test_outus=# SELECT pg_size_pretty(pg_TABLE_size('test'));
 pg_size_pretty 
----------------
 35 MB
(1 row)
```
-----------------------  
* 5 раз обновить все строчки и добавить к каждой строчке любой символ
```sql
CREATE PROCEDURE update_test()
LANGUAGE plpgsql
AS $$
 declare step int := 1;
 begin
  while step <= 5 loop
  raise notice 'Step %',step;
  UPDATE test SET c1 =1;
  
   step := step+1;
  end loop;
 end;
 $$;
```

CALL update_test();


--------------------------------
* Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```sql  
SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test';

 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |    5000000 |    499 | 2024-01-06 15:19:58.183714+03
(1 row)

```
----------------------
* Подождать некоторое время, проверяя, пришел ли автовакуум
```sql
SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test';

 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |          0 |      0 | 2024-01-06 15:22:06.632755+03
(1 row)
```
----------------------  
* 5 раз обновить все строчки и добавить к каждой строчке любой символ
```sql
test_outus=# call update_test();
NOTICE:  Step 1
NOTICE:  Step 2
NOTICE:  Step 3
NOTICE:  Step 4
NOTICE:  Step 5

```
--------------------------------  
* Посмотреть размер файла с таблицей
```sql
test_outus=# SELECT pg_size_pretty(pg_TABLE_size('test'));
 pg_size_pretty 
----------------
 208 MB
(1 row)

```
--------------------------------  
* Отключить Автовакуум на конкретной таблице
```sql
 ALTER TABLE test SET (autovacuum_enabled = off);

```
------------------------------  
* 10 раз обновить все строчки и добавить к каждой строчке любой символ

  ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/f3334df4-6109-4f77-abd3-c43089f20b4d)
  

-------------------
  
* Посмотреть размер файла с таблицей

```sql
test_outus=# SELECT pg_size_pretty(pg_TABLE_size('test'));
 pg_size_pretty 
----------------
 553 MB
(1 row)

```
  -------------------
* Объясните полученный результат

Так как после обновления таблицы был отключен автовакуум, в таблице остались мертвые 5мл записей, автовакуум не почистил мертвые записи, после того как еще раз таблица test была обновлена 10 раз, к 5мл мертвых записей добавились еще 10 мл мертвых записей и размер таблицы увеличился. Если бы автовакуум был включен, тогда в таблице не было 5мл мертвых записей и пространство было бы освобождено для новых записей и таблица не увеличилась так сильно в размере. 
  
* Не забудьте включить автовакуум)

 ALTER TABLE test SET (autovacuum_enabled = on);  

---------------------
  
* Задание со *:
* Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.

```sql
CREATE PROCEDURE update_test3()
LANGUAGE plpgsql
AS $$
 declare step int := 1;
 begin
  while step <= 10 loop
  raise notice 'Step %',step;
  UPDATE test SET c1 =100000;
  
   step := step+1;
  end loop;
 end;
 $$;
```
-----------------------
