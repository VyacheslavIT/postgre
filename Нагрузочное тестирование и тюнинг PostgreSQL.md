* развернуть виртуальную машину любым удобным способом



  
------------------------------------------------------
* поставить на неё PostgreSQL 15 любым способом

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/9f038f6b-264b-419a-9ab2-257ec9b0d119)

-----------------------------------------------------
* настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины

Выставил настройки из pg_tune

```sql

max_connections = 100
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 2621kB
huge_pages = off
min_wal_size = 1GB
max_wal_size = 4GB

```

и асинхронный режим

----------------------------------------------------
* нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)

  подготовка pgbench -i -p 5433 -s 10
  

Тест 1: в синхронном режиме с настройка по умолчанию pgbench -c 50 -j 2 -P 10 -p 5433 -T 60 -U postgres postgres

tps = 3500.993491

Тест 2: в синхронном режиме настройки с  pgtune

tps = 3304.568552

```sql
postgres=# show synchronous_commit;
 synchronous_commit 
--------------------
 off
(1 row)

```
Тест 3 режим асинхронный настройки  по умолчанию  pgbench -c 50 -j 2 -P 10 -p 5433 -T 60 -U postgres postgres

tps = 4302.523401

Тест 4 режим асинхронный настройки с  pgtune  pgbench -c 50 -j 2 -P 10 -p 5433 -T 60 -U postgres postgres

tps = 4325.773212 



-----------------------------------------------------
* написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему
  
tps = 4325.773212 
```sql
max_connections = 100
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 2621kB
huge_pages = off
min_wal_size = 1GB
max_wal_size = 4GB
```

----------------------------------------------------
* Задание со *:
* аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)




1:  установка curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench
```sql

 CREATE USER sbtest WITH PASSWORD 'password';
 
 CREATE DATABASE sbtest;
 
GRANT ALL PRIVILEGES ON DATABASE sbtest TO sbtest;
```

2: подготовим 100 таблиц по 10000 строк : sysbench --db-driver=pgsql  --tables=100   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 /usr/share/sysbench/oltp_read_write.lua prepare


3: Запустим тест sysbench --db-driver=pgsql  --tables=100   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 -threads=2 /usr/share/sysbench/oltp_read_write.lua run


```sql
Тест 1: в синхронном режиме с настройка по умолчанию

Initializing worker threads...

Threads started!

[ 2s ] thds: 1 tps: 701.11 qps: 14031.61 (r/w/o: 9822.48/2806.42/1402.71) lat (ms,95%): 2.48 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 1 tps: 863.52 qps: 17270.45 (r/w/o: 12089.32/3454.09/1727.05) lat (ms,95%): 1.73 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 1 tps: 830.18 qps: 16594.68 (r/w/o: 11615.58/3318.74/1660.37) lat (ms,95%): 1.89 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 1 tps: 866.58 qps: 17340.51 (r/w/o: 12139.06/3468.30/1733.15) lat (ms,95%): 1.86 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 1 tps: 873.81 qps: 17476.12 (r/w/o: 12233.28/3495.22/1747.61) lat (ms,95%): 1.89 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            115808
        write:                           33088
        other:                           16544
        total:                           165440
    transactions:                        8272   (826.91 per sec.)
    queries:                             165440 (16538.28 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0023s
    total number of events:              8272

Latency (ms):
         min:                                    0.86
         avg:                                    1.21
         max:                                   15.96
         95th percentile:                        1.96
         sum:                                 9988.98

Threads fairness:
    events (avg/stddev):           8272.0000/0.00
    execution time (avg/stddev):   9.9890/0.00



```

```sql
Тест 2: в синхронном режиме настройки с  pgtune

Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 829.01 qps: 16591.77 (r/w/o: 11616.69/3316.06/1659.03) lat (ms,95%): 5.57 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 871.57 qps: 17436.36 (r/w/o: 12204.95/3488.27/1743.14) lat (ms,95%): 3.25 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 1006.00 qps: 20117.44 (r/w/o: 14081.46/4023.99/2011.99) lat (ms,95%): 3.13 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 964.50 qps: 19288.01 (r/w/o: 13503.01/3856.00/1929.00) lat (ms,95%): 3.19 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 1011.46 qps: 20232.17 (r/w/o: 14162.92/4046.33/2022.92) lat (ms,95%): 2.97 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            131152
        write:                           37472
        other:                           18736
        total:                           187360
    transactions:                        9368   (936.45 per sec.)
    queries:                             187360 (18729.00 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0028s
    total number of events:              9368

Latency (ms):
         min:                                    0.95
         avg:                                    2.13
         max:                                   13.04
         95th percentile:                        3.36
         sum:                                19986.16

Threads fairness:
    events (avg/stddev):           4684.0000/259.00
    execution time (avg/stddev):   9.9931/0.00



```




```sql
Тест 3 режим асинхронный настройки с pgtune

Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 1063.88 qps: 21281.06 (r/w/o: 14896.79/4255.51/2128.76) lat (ms,95%): 3.43 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 1372.57 qps: 27460.81 (r/w/o: 19225.42/5490.26/2745.13) lat (ms,95%): 2.14 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 1269.03 qps: 25381.00 (r/w/o: 17766.35/5076.60/2538.05) lat (ms,95%): 2.35 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 1312.99 qps: 26257.28 (r/w/o: 18378.35/5252.96/2625.98) lat (ms,95%): 2.61 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 1207.49 qps: 24145.32 (r/w/o: 16902.38/4828.46/2414.48) lat (ms,95%): 2.30 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            174356
        write:                           49816
        other:                           24908
        total:                           249080
    transactions:                        12454  (1244.89 per sec.)
    queries:                             249080 (24897.77 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0033s
    total number of events:              12454

Latency (ms):
         min:                                    0.65
         avg:                                    1.60
         max:                                   15.00
         95th percentile:                        2.43
         sum:                                19983.78

Threads fairness:
    events (avg/stddev):           6227.0000/210.00
    execution time (avg/stddev):   9.9919/0.00


```
```sql
Тест 3 режим асинхронный настройки  по умолчанию

Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 1124.88 qps: 22506.53 (r/w/o: 15756.27/4499.51/2250.75) lat (ms,95%): 3.30 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 1865.60 qps: 37311.56 (r/w/o: 26117.44/7463.41/3730.71) lat (ms,95%): 2.30 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 1408.99 qps: 28180.85 (r/w/o: 19727.40/5634.97/2818.49) lat (ms,95%): 2.39 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 2365.01 qps: 47305.77 (r/w/o: 33113.69/9461.55/4730.53) lat (ms,95%): 1.39 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 2390.98 qps: 47813.53 (r/w/o: 33469.67/9561.91/4781.95) lat (ms,95%): 1.25 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            256396
        write:                           73255
        other:                           36629
        total:                           366280
    transactions:                        18314  (1830.75 per sec.)
    queries:                             366280 (36615.09 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0026s
    total number of events:              18314

Latency (ms):
         min:                                    0.63
         avg:                                    1.09
         max:                                   12.19
         95th percentile:                        2.35
         sum:                                19979.54

Threads fairness:
    events (avg/stddev):           9157.0000/401.00
    execution time (avg/stddev):   9.9898/0.00


```
