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

> ![image](https://github.com/VyacheslavIT/postgre/assets/136000255/ad7c1fa3-35f9-4eee-871f-35ff2304ddcd)

--------------------------------
* попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start

* напишите получилось или нет и почему

>Кластер не получилось запустить, появилась ошибка, так как мы переместили data файлы в другую директорию.

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

* зайдите через через psql и проверьте содержимое ранее созданной таблицы

>Кластер запустился

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/e369ade6-211b-41de-b43d-99004465e68a)

```sql
postgres=# select * from test;
 c1 
----
 1
(1 row)
```
таблица на месте
```sql
postgres=# show data_directory;
  data_directory   
-------------------
 /mnt/data/15/main
(1 row)
```
новый путь 

---------------------------

задание со звездочкой *: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.

>На первой машине остановил кластер и отмонтировал диск, в настройках первой виртуальной машины отсоединил данный диск.
>На второй виртуальной машине добавил данный диск и включил машину, далее подмонтировал данный диск в  созданную папку data в директории mnt.
>Дал права postgres на папку data, остановил кластер, в файле postgresql.conf указал новый путь к data файлу, удалил содержимое из директориии /var/lib/postgres,запустил кластер.


>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/4da1d00c-f628-4aa4-ae13-61c0055152b2)


>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/9f8cfd37-2601-4f78-b8e8-1c49537f347c)




>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/d5c75a2e-d422-48f7-8568-22e08dbb5c9c)

```sql
postgres=# select * from test;
 c1 
----
 1
(1 row)

postgres=# 
```
```sql
postgres=# show data_directory;
  data_directory   
-------------------
 /mnt/data/15/main
(1 row)

postgres=# 
```
