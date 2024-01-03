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
postgres=# \c tsetdb
connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  database "tsetdb" does not exist
Previous connection kept
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# 
``` 
