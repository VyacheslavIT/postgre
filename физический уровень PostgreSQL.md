* создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
* поставьте на нее PostgreSQL 15 через sudo apt
* проверьте что кластер запущен через sudo -u postgres pg_lsclusters
  
*pg_lsclusters*

>Ver Cluster Port Status Owner    Data directory              Log file

>15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/7e28e658-2458-4e38-9b5e-72c03865d6bd)
-----------------------------
* зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
  
*$  sudo -u postgres psql*
  
* postgres=# create table test(c1 text);
* postgres=# insert into test values('1');
* \q
```sql
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres=# select * from test;
 c1 
----
 1
(1 row)

postgres=#
```
-------------------------
* остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/7f972d4c-8ef0-4156-8fb4-7cf2280347eb)
  
--------------------------

* создайте новый диск к ВМ размером 10GB

*ls -l |grep sd*

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/a68a8677-b8ee-418b-9c1f-f601c0e0fb02)

