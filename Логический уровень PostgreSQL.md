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


