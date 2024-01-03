* создайте новый кластер PostgresSQL 14
  
>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/8a1ff850-efb8-4d33-b188-77236f5280d4)

* зайдите в созданный кластер под пользователем postgres
  
  *sudo -u postgres psql*
  
*   создайте новую базу данных testdb

``` sql
create database testdb;
```


```sql
postgres=# \l
                                                 List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | 
(4 rows)
```

* зайдите в созданную базу данных под пользователем postgres
  
```sql
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# 
``` 

* создайте новую схему testnm
  
```sql
testdb=# create schema testnm;
CREATE SCHEMA
testdb=# select * from pg_namespace;
  oid  |      nspname       | nspowner |                            nspacl                             
-------+--------------------+----------+---------------------------------------------------------------
    99 | pg_toast           |       10 | 
    11 | pg_catalog         |       10 | {postgres=UC/postgres,=U/postgres}
  2200 | public             |     6171 | {pg_database_owner=UC/pg_database_owner,=U/pg_database_owner}
 13243 | information_schema |       10 | {postgres=UC/postgres,=U/postgres}
 16389 | testnm             |       10 | 
(5 rows)
```

* создайте новую таблицу t1 с одной колонкой c1 типа integer
```sql
CREATE TABLE t1(c1 integer);
CREATE TABLE
testdb=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)
```
* вставьте строку со значением c1=1
  
```sql
INSERT INTO t1 values(1);
INSERT 0 1
testdb=# select * from t1;
 c1 
----
  1
(1 row)
```
* создайте новую роль readonly


  
```sql
create role readonly;
CREATE ROLE

SELECT * FROM pg_roles where rolname = 'readonly';
 rolname  | rolsuper | rolinherit | rolcreaterole | rolcreatedb | rolcanlogin | rolreplication | rolconnlimit | rolpassword | rolvaliduntil | rolbypassrls | rolconfig |  oid  
----------+----------+------------+---------------+-------------+-------------+----------------+--------------+-------------+---------------+--------------+-----------+-------
 readonly | f        | t          | f             | f           | t           | f              |           -1 | ********    |               | f            |           | 16393
(1 row)


```
* дайте новой роли право на подключение к базе данных testdb
  
```sql
GRANT CONNECT ON DATABASE testdb TO readonly;
GRANT
```
* дайте новой роли право на использование схемы testnm
  
```sql  
GRANT USAGE ON SCHEMA testnm TO readonly;
GRANT
```

* дайте новой роли право на select для всех таблиц схемы testnm

```sql
GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
```
* создайте пользователя testread с паролем test123

```sql
create user testread password 'test123';
CREATE ROLE

SELECT * FROM pg_user where usename = 'testread';
 usename  | usesysid | usecreatedb | usesuper | userepl | usebypassrls |  passwd  | valuntil | useconfig 
----------+----------+-------------+----------+---------+--------------+----------+----------+-----------
 testread |    16394 | f           | f        | f       | f            | ******** |          | 
(1 row)



``` 
  
* дайте роль readonly пользователю testread
  
```sql
testdb=# grant readonly to testread;
GRANT ROLE
```

* зайдите под пользователем testread в базу данных testdb

 ```sql
psql -h localhost -d testdb -U testread;
Password for user testread: 
psql (15.5 (Ubuntu 15.5-1.pgdg23.10+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

testdb=> 
```

* сделайте select * from t1;
  
```sql
testdb=>  select * from t1;
ERROR:  permission denied for table t1
testdb=> 
```
получилось? 

>Нет

напишите что именно произошло в тексте домашнего задания
```sql
ERROR:  permission denied for table t1
```
у вас есть идеи почему? ведь права то дали?


```sql
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)
```
Данная таблица расположена в схеме public, права мы выдавали на схему testnm, поэтому и нет прав чтения данной таблицы.

* посмотрите на список таблиц
```sql
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)
```

* а почему так получилось с таблицей 
  
Нет пользовательской схемы, а так как мы явно не указывали схему по умолчанию табличка попала в схему public.

```sql 
postgres=# SELECT current_schemas(true);
   current_schemas   
---------------------
 {pg_catalog,public}
(1 row)
```
 
```sql
postgres=# SHOW search_path;
   search_path   
-----------------
 "$user", public
(1 row)

```
 
* вернитесь в базу данных testdb под пользователем postgres

```sql
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# select * from t1;
 c1 
----
  1
(1 row)

testdb=#
```
* удалите таблицу t1
  
```sql
testdb=# drop table t1;
DROP TABLE
testdb=# select * from t1;
ERROR:  relation "t1" does not exist
LINE 1: select * from t1;
                      ^
testdb=# 


``` 
* создайте ее заново но уже с явным указанием имени схемы testnm
  
* вставьте строку со значением c1=1

```sql
testdb=# CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE
testdb=# INSERT INTO testnm.t1 values(1);
INSERT 0 1

testdb=# select * from testnm.t1;
 c1 
----
  1
(1 row)
```

* зайдите под пользователем testread в базу данных testdb
  
* сделайте select * from testnm.t1;
  
* получилось?
  
```sql
select * from testnm.t1;
ERROR:  permission denied for table t1
``` 
* есть идеи почему? 

Ранее права были выданы для уже созданных объектов в схеме, таблицу t1 мы пересоздали.

* как сделать так чтобы такое больше не повторялось?

Сначала снова выполняем GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly для существующих таблиц, после выполнения появятся права на выполнение запроса select из таблицы t1, 
Чтобы в дальнейшим видеть вновь созданные таблицы выполняем ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly;

* сделайте select * from testnm.t1;
```sql
  testdb=> select * from testnm.t1;
 c1 
----
  1
(1 row)
```
* теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
  
testdb=> create table t2(c1 integer);
ERROR:  permission denied for schema public

Использовал 15 версию postgresql.
В PostgreSQL 15 права на создание таблиц (CREATE TABLE) по умолчанию отозваны у роли PUBLIC. Роли PUBLIC оставлено только право на использование объектов (USAGE). Это изменение призвано повысить безопасность базы данных и предотвратить случайное создание ненужных таблиц. Если  нужно создать таблицу, нужно предоставить себе соответствующие права, либо использовать другую роль с необходимыми правами.
----------------------

Установил 14 версию 

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/505b0bc4-01da-4054-8b05-67bf7e264d7d)

*теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
*а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?




```sql
psql -h localhost -p 5433 -d testdb -U testread;
Password for user testread: 
psql (15.5 (Ubuntu 15.5-1.pgdg23.10+1), server 14.10 (Ubuntu 14.10-1.pgdg23.10+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

testdb=> create table t2(c1 integer);
CREATE TABLE
testdb=> insert into t2 values (2);
INSERT 0 1
testdb=> 
```
Так как явно не указаывали схему по умолчанию ,таблица создались в первой схеме public.
Роли public предоставляются права на использование всех объектов в схеме public, включая возможность создания новых объектов. 
Также схема public по умолчанию создается в каждой новой базе данных в PostgreSQL.


