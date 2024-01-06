
* Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
* Установить на него PostgreSQL 15 с дефолтными настройками
* Создать БД для тестов: выполнить pgbench -i postgres
* Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
* Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
* Протестировать заново
* Что изменилось и почему?
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
  
* Посмотреть размер файла с таблицей
```sql
test_outus=# SELECT pg_size_pretty(pg_TABLE_size('test'));
 pg_size_pretty 
----------------
 208 MB
(1 row)

```
  
* Отключить Автовакуум на конкретной таблице
```sql
 ALTER TABLE test SET (autovacuum_enabled = off);

```
  
* 10 раз обновить все строчки и добавить к каждой строчке любой символ

  ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/f3334df4-6109-4f77-abd3-c43089f20b4d)
  
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
* Не забудьте включить автовакуум)
* Задание со *:
* Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.
