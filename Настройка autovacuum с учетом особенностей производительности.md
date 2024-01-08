
* Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
* Установить на него PostgreSQL 15 с дефолтными настройками
  
![image](https://github.com/VyacheslavIT/postgre/assets/136000255/7b1aebe5-1948-45aa-82c1-ff49fcf23e47)


 ------------------- 
* Создать БД для тестов: выполнить pgbench -i postgres

test_outus=# create database pgbench;
CREATE DATABASE
test_outus=# \q
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

pgbench=# \dt
              List of relations
 Schema |       Name       | Type  |  Owner   
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
(4 rows)


 ------------------ 
* Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
  
ostgres@slavavm1:/home/slava$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.5 (Ubuntu 15.5-1.pgdg23.10+1))
starting vacuum...end.
progress: 6.0 s, 594.0 tps, lat 13.403 ms stddev 9.552, 0 failed
progress: 12.0 s, 801.9 tps, lat 9.982 ms stddev 5.898, 0 failed
progress: 18.0 s, 861.9 tps, lat 9.283 ms stddev 5.367, 0 failed
progress: 24.0 s, 903.2 tps, lat 8.854 ms stddev 5.060, 0 failed
progress: 30.0 s, 887.7 tps, lat 9.013 ms stddev 5.168, 0 failed
progress: 36.0 s, 907.5 tps, lat 8.817 ms stddev 4.916, 0 failed
progress: 42.0 s, 898.1 tps, lat 8.904 ms stddev 5.048, 0 failed
progress: 48.0 s, 907.5 tps, lat 8.815 ms stddev 5.037, 0 failed
progress: 54.0 s, 876.0 tps, lat 9.129 ms stddev 5.252, 0 failed
progress: 60.0 s, 702.0 tps, lat 11.397 ms stddev 7.363, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 50047
number of failed transactions: 0 (0.000%)
latency average = 9.589 ms
latency stddev = 5.974 ms
initial connection time = 20.202 ms
tps = 834.123173 (without initial connection time)
postgres@slavavm1:/home/slava$ 


  
-------------------  
* Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
* Протестировать заново
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
