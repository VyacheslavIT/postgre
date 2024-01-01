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

* добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)

*ls -l | grep sd*

*mount | grep sdb*

*cat /etc/fstab*

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/a68a8677-b8ee-418b-9c1f-f601c0e0fb02)

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/1338dcd4-b1c4-4444-a2c6-75d10af3f450)

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/6bba6047-46aa-46c6-8621-b951296e2481)
-------------------------------
* сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

*sudo chown postgres -R /mnt/data*

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/4c362ada-b85f-4622-8cfd-53c5a04a9dcd)
------------------------------
* перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/eba31f3a-ebf3-4ba4-9dee-f27ff1ce0577)

root@vm1:/mnt/data/main# ls -l

total 80

drwx------ 6 postgres postgres 4096 Dec 25 21:51 base

drwx------ 2 postgres postgres 4096 Dec 25 21:52 global

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_commit_ts

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_dynshmem

drwx------ 4 postgres postgres 4096 Dec 25 22:01 pg_logical

drwx------ 4 postgres postgres 4096 Dec 24 19:44 pg_multixact

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_notify

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_replslot

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_serial

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_snapshots

drwx------ 2 postgres postgres 4096 Dec 25 22:01 pg_stat

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_stat_tmp

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_subtrans

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_tblspc

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_twophase

-rw------- 1 postgres postgres    3 Dec 24 19:44 PG_VERSION

drwx------ 3 postgres postgres 4096 Dec 24 19:44 pg_wal

drwx------ 2 postgres postgres 4096 Dec 24 19:44 pg_xact

-rw------- 1 postgres postgres   88 Dec 24 19:44 postgresql.auto.conf

-rw------- 1 postgres postgres  130 Dec 25 20:51 postmaster.opts

--------------------------------
* попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start

* напишите получилось или нет и почему

>Кластер не получилось запустить, появилась ошибка, так как мы переместили файлы БД в другую деректорию.

> slava@vm1:~$ sudo -u postgres pg_ctlcluster 15 main start

> Error: /var/lib/postgresql/15/main is not accessible or does not exist

------------------------------------------

* задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его

* напишите что и почему поменяли

> Указал новый путь в файле postgresql.conf

> В разделе File Location, прописал новый путь data_directory куда перенесли файлы БД.  

------------------------------------------
* попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start

* напишите получилось или нет и почему

>Кластер запустился

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/e369ade6-211b-41de-b43d-99004465e68a)



